# Jenkins Upstream, Downstream, and Parallel Execution

## 1. Upstream and Downstream Jobs

**Definition:** An upstream job triggers a downstream job after a specific event, such as a successful build.

**Example workflow:**

`DEV → STG → UAT`

- DEV runs successfully, then STG starts.
- STG runs successfully, then UAT starts.
- If a job fails, the next job should not start.

### 1.1 DEV Job — Trigger STG After Success

```groovy
pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'anil-free',
                    url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Report') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Trigger STG') {
            steps {
                build job: 'swiss-stg',
                      wait: true,
                      propagate: true
            }
        }
    }
}
```

**Explanation:** The DEV job triggers `swiss-stg` and waits for it to finish. If STG fails, the DEV pipeline is marked as failed too.

### 1.2 STG Job — Trigger UAT After Success

Add the following stage to the STG pipeline after its required build and testing stages:

```groovy
stage('Trigger UAT') {
    steps {
        build job: 'swiss-uat',
              wait: true,
              propagate: true
    }
}
```

**Explanation:** The UAT job starts only after the STG pipeline reaches this stage. With `wait: true` and `propagate: true`, the triggering pipeline waits for UAT and reflects its result.

### 1.3 UAT Job

Configure `swiss-uat` with the required checkout, build, testing, and deployment stages.

**Note:** To make the sequence strictly DEV success → STG success → UAT success, each upstream job must trigger the next job only after its own required stages succeed.

---

## 2. Parallel Execution — Maven Build and SonarQube

**Definition:** Parallel execution runs independent tasks simultaneously to reduce total pipeline execution time.

```groovy
pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'anil-free',
                    url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
            }
        }

        stage('Maven Build and SonarQube') {
            parallel {
                stage('Maven Build') {
                    steps {
                        sh 'mvn clean package'
                    }
                }

                stage('SonarQube Report') {
                    steps {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
    }
}
```

**Important:** These two tasks may run simultaneously, but this example does not guarantee that the SonarQube scan uses the output of the Maven build. If the scan requires compiled classes or test reports, run the build first and then run SonarQube analysis, or configure a suitable shared build workflow.

---

## 3. Parallel Execution — Nexus Upload and Tomcat Deployment

**Definition:** Nexus upload and Tomcat deployment can run in parallel when both use the same already-built artifact and neither depends on the other's result.

**Recommended workflow:** Checkout → Build → Tests/SonarQube → Parallel Nexus Upload and Tomcat Deployment.

```groovy
pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'anil-free',
                    url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Nexus Upload and Tomcat Deployment') {
            parallel {
                stage('Upload to Nexus') {
                    steps {
                        sh 'mvn deploy'
                    }
                }

                stage('Deploy to Tomcat') {
                    steps {
                        withCredentials([
                            usernamePassword(
                                credentialsId: 'tomcat-manager-credentials',
                                usernameVariable: 'TOMCAT_USER',
                                passwordVariable: 'TOMCAT_PASSWORD'
                            )
                        ]) {
                            sh '''
                                curl --fail --silent --show-error \
                                -u "$TOMCAT_USER:$TOMCAT_PASSWORD" \
                                --upload-file target/maven-web-application.war \
                                "http://<TOMCAT-IP>:9090/manager/text/deploy?path=/maven-web-application&update=true"
                            '''
                        }
                    }
                }
            }
        }
    }
}
```

### Important Notes

1. Replace `<TOMCAT-IP>` with the correct Tomcat server IP.
2. Create the Jenkins username/password credential with ID `tomcat-manager-credentials`.
3. Configure Nexus repository details in `pom.xml` and authentication in Maven `settings.xml`.
4. Ensure the Tomcat Manager application allows deployment requests from Jenkins.
5. Both parallel stages must have sufficient access to the same built WAR file.
6. If Nexus publishing must succeed before deployment, do not run those two tasks in parallel; use separate sequential stages instead.
7. Use Jenkins Declarative Pipeline's `parallel { stage(...) { ... } }` syntax for parallel stages.

---

## 4. Upstream/Downstream vs. Parallel Execution

| Feature | Upstream/Downstream | Parallel Execution |
|---|---|---|
| Execution | One job triggers another | Independent tasks run simultaneously |
| Main purpose | Control job sequence | Reduce execution time |
| Example | DEV → STG → UAT | Build and independent checks |
| Dependency | Usually sequential | Tasks should be independent |
| Failure handling | Can prevent later jobs from starting | A failed branch normally fails the parallel stage |

## 5. Final Summary

- **Upstream/Downstream:** Use `build job: 'job-name', wait: true, propagate: true` when the next job should start and its result should affect the calling pipeline.
- **Parallel execution:** Use Declarative `parallel` stages for independent tasks.
- **Best practice:** Build the artifact once, run required quality checks, and then publish or deploy it according to the dependency order.
