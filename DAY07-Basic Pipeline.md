# Jenkins Scripted Pipeline Using Groovy — Practical CI/CD Guide

## 1. Basic Scripted Pipeline Syntax

Jenkins Scripted Pipeline uses Groovy-based syntax to automate CI/CD tasks. The `node` block allocates an executor and workspace, while `stage` organizes the pipeline into logical steps.

### Example

```groovy
node {
    stage('Stage One') {
        sh 'echo "Hello One"'
    }

    stage('Stage Two') {
        sh 'echo "Hello Two"'
    }

    stage('Stage Three') {
        sh 'echo "Hello Three"'
    }
}
```

### Explanation

- `node { }` — Allocates a Jenkins executor and workspace.
- `stage('Stage One')` — Creates a named pipeline stage.
- `sh` — Executes a shell command on a Linux agent.
- `echo` — Prints a message to the Jenkins console.

**Expected output:**

```text
Hello One
Hello Two
Hello Three
```

---

## 2. Git Checkout Using Scripted Pipeline

### Objective

Download source code from GitHub into the Jenkins workspace.

### Pipeline code

```groovy
node {
    stage('Git Checkout') {
        git branch: 'anil-test',
            url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
    }
}
```

### Explanation

- `git` — Checks out source code using the Jenkins Git plugin.
- `branch` — Specifies the branch to check out.
- `url` — Specifies the Git repository URL.
- The source code is downloaded into the Jenkins workspace.

### How to configure it in Jenkins

1. Open Jenkins Dashboard.
2. Click **New Item**.
3. Enter a job name, for example `scripted-pipeline-practice`.
4. Select **Pipeline** and click OK.
5. Scroll to the Pipeline section.
6. Select **Pipeline script**.
7. Paste the Groovy script.
8. Click **Save**, then **Build Now**.
9. Open **Console Output** to verify the checkout.

If the repository is private, configure Jenkins credentials and use them in the Git checkout step.

---

## 3. Maven Build Using Scripted Pipeline

### Objective

Compile the Java application, run tests, and generate a JAR or WAR artifact.

### Prerequisites

- Maven must be installed or configured in Jenkins.
- A compatible JDK must be available.
- The Git checkout stage must run before the build.
- The project must contain a valid `pom.xml`.

### Pipeline code

```groovy
node {
    stage('Git Checkout') {
        git branch: 'anil-test',
            url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
    }

    def mavenHome = tool name: 'maven-3.9.9'

    stage('Maven Build') {
        sh "${mavenHome}/bin/mvn clean package"
    }
}
```

### Explanation

- `tool` locates the Maven installation configured in Jenkins.
- `mvn clean package` removes old build output, compiles the code, runs configured tests, and packages the application.
- The generated artifact is usually available inside the project's `target/` directory.

### Configure Maven in Jenkins

1. Open **Manage Jenkins → Tools**.
2. Find **Maven installations**.
3. Add a Maven installation.
4. Set the installation name to `maven-3.9.9`.
5. Save the configuration.

The name in the pipeline must match the configured installation name.

### Verify the build

```bash
ls -lh target/
```

Example artifact:

```text
target/maven-web-application.war
```

The actual artifact name depends on the project's `pom.xml`.

---

## 4. SonarQube Analysis Using Scripted Pipeline

### Objective

Analyze source code for code quality issues, bugs, and security-related findings using SonarQube.

### Prerequisites

- Git checkout must be completed.
- Maven build prerequisites must be satisfied.
- SonarQube must be configured in Jenkins.
- The SonarQube server must be reachable from the build agent.
- Authentication and the SonarQube Maven scanner configuration must be correct.

### Pipeline code

```groovy
node {
    stage('Git Checkout') {
        git branch: 'anil-test',
            url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
    }

    def mavenHome = tool name: 'maven-3.9.9'

    stage('Maven Build') {
        sh "${mavenHome}/bin/mvn clean package"
    }

    stage('SonarQube Report') {
        withSonarQubeEnv('SonarQubeServer') {
            sh "${mavenHome}/bin/mvn sonar:sonar"
        }
    }
}
```

### Explanation

- `withSonarQubeEnv` supplies the configured SonarQube connection details.
- `SonarQubeServer` must match the SonarQube server configuration name in Jenkins.
- `sonar:sonar` invokes the SonarQube Maven goal.
- The analysis is sent to the configured SonarQube server.

### Important note

The Maven SonarQube plugin and a compatible Java/Maven setup must be available. Configure the SonarQube token securely in Jenkins rather than hardcoding it in the pipeline.

For quality-gate enforcement, configure the appropriate SonarQube webhook and Jenkins quality-gate step.

---

## 5. Deploy the WAR File to Tomcat

### Objective

Deploy the generated WAR file to a running Tomcat server using the Tomcat Manager text interface.

### Prerequisites

- Git checkout and Maven build must succeed.
- Tomcat must be running.
- The WAR file must exist.
- Tomcat Manager's text interface must be enabled and accessible.
- The deployment user must have the appropriate Manager role.
- The Tomcat server's network rules must permit Jenkins to connect.

### Pipeline code

```groovy
node {
    stage('Git Checkout') {
        git branch: 'anil-test',
            url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
    }

    def mavenHome = tool name: 'maven-3.9.9'

    stage('Maven Build') {
        sh "${mavenHome}/bin/mvn clean package"
    }

    stage('Deploy to Tomcat') {
        withCredentials([
            usernamePassword(
                credentialsId: 'tomcat-manager-creds',
                usernameVariable: 'TOMCAT_USER',
                passwordVariable: 'TOMCAT_PASSWORD'
            )
        ]) {
            sh '''
                curl --fail --show-error --silent \
                  --user "$TOMCAT_USER:$TOMCAT_PASSWORD" \
                  --upload-file target/maven-web-application.war \
                  "http://3.109.2.100:9090/manager/text/deploy?path=/maven-web-application&update=true"
            '''
        }
    }
}
```

### Explanation

- `withCredentials` retrieves the Tomcat username and password from Jenkins Credentials.
- `--upload-file` uploads the WAR file to the Tomcat Manager deployment endpoint.
- `path=/maven-web-application` specifies the application's context path.
- `update=true` requests an update if the application is already deployed.
- `--fail` makes curl return a failure for HTTP error responses.

### Configure Tomcat credentials in Jenkins

1. Open **Manage Jenkins → Credentials**.
2. Add a username/password credential.
3. Set the ID to `tomcat-manager-creds`.
4. Store the authorized Tomcat Manager username and password.
5. Save the credential.

Replace the example server IP and port with your actual Tomcat address. Use HTTPS where possible, especially when credentials cross an untrusted network.

---

## 6. Publish the Build Artifact to Nexus Repository

### Objective

Publish a versioned Maven artifact to Nexus Repository Manager.

### Prerequisites

- Git checkout must succeed.
- Maven must be configured.
- Nexus must be reachable.
- The project must have appropriate artifact coordinates.
- `pom.xml` must contain a valid `distributionManagement` configuration.
- Maven credentials must be configured securely in `settings.xml`.

### Pipeline code

```groovy
node {
    stage('Git Checkout') {
        git branch: 'anil-test',
            url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
    }

    def mavenHome = tool name: 'maven-3.9.9'

    stage('Maven Build') {
        sh "${mavenHome}/bin/mvn clean package"
    }

    stage('Backup to Nexus') {
        sh "${mavenHome}/bin/mvn deploy"
    }
}
```

### Why use `mvn deploy`?

The Maven `deploy` phase runs the preceding default lifecycle phases, including packaging and installation, and then publishes the artifact to the configured remote Maven repository.

**Important:** `mvn deploy` does not automatically upload an artifact to Nexus unless the Maven project and repository credentials are configured correctly.

### Example `distributionManagement`

Add the appropriate configuration to `pom.xml`:

```xml
<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <url>https://nexus.example.com/repository/maven-releases/</url>
    </repository>

    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>https://nexus.example.com/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

Replace the example URLs with your actual Nexus repository URLs.

The repository IDs must match the corresponding server IDs in Maven's `settings.xml`. Configure credentials securely on the Jenkins agent or through an approved Jenkins configuration mechanism.

---

## 7. Complete Scripted Pipeline: Git + Maven + SonarQube + Nexus + Tomcat

This example combines all five stages into a single CI/CD pipeline for practice.

### Complete pipeline code

```groovy
node {
    stage('Git Checkout') {
        git branch: 'anil-test',
            url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
    }

    def mavenHome = tool name: 'maven-3.9.9'

    stage('Maven Build') {
        sh "${mavenHome}/bin/mvn clean package"
    }

    stage('SonarQube Report') {
        withSonarQubeEnv('SonarQubeServer') {
            sh "${mavenHome}/bin/mvn sonar:sonar"
        }
    }

    stage('Backup to Nexus') {
        sh "${mavenHome}/bin/mvn deploy"
    }

    stage('Deploy to Tomcat') {
        withCredentials([
            usernamePassword(
                credentialsId: 'tomcat-manager-creds',
                usernameVariable: 'TOMCAT_USER',
                passwordVariable: 'TOMCAT_PASSWORD'
            )
        ]) {
            sh '''
                test -f target/maven-web-application.war

                curl --fail --show-error --silent \
                  --user "$TOMCAT_USER:$TOMCAT_PASSWORD" \
                  --upload-file target/maven-web-application.war \
                  "http://3.109.2.100:9090/manager/text/deploy?path=/maven-web-application&update=true"
            '''
        }
    }
}
```

### Pipeline execution flow

```text
GitHub Repository
        |
        v
   Git Checkout
        |
        v
    Maven Build
        |
        v
  SonarQube Analysis
        |
        v
  Publish Artifact to Nexus
        |
        v
   Deploy WAR to Tomcat
        |
        v
 Verify Application
```

### Important considerations

1. Configure the Maven installation with the name `maven-3.9.9`.
2. Configure the SonarQube server in Jenkins with the name `SonarQubeServer`.
3. Configure the Tomcat credentials with the ID `tomcat-manager-creds`.
4. Configure Nexus repositories and credentials in Maven.
5. Confirm the WAR file name matches the actual build artifact.
6. Confirm the Tomcat IP address, port, and Manager endpoint.
7. Keep secrets out of the pipeline source code.
8. Consider adding a SonarQube quality-gate check before publishing and deploying.
9. For production, use controlled release versions and deploy the same tested artifact rather than unnecessarily rebuilding it at each stage.

---

## 8. Common Jenkins Scripted Pipeline Commands

| Requirement | Command or step |
|---|---|
| Print a message | `echo 'Hello Jenkins'` |
| Execute a Linux command | `sh 'pwd'` |
| Clone a Git branch | `git branch: 'anil-test', url: 'REPOSITORY_URL'` |
| Configure Maven | `tool name: 'maven-3.9.9'` |
| Build a Maven project | `mvn clean package` |
| Run SonarQube analysis | `mvn sonar:sonar` |
| Publish a Maven artifact | `mvn deploy` |
| Upload a WAR to Tomcat | `curl --upload-file ...` |
| Use Jenkins credentials | `withCredentials([...])` |
| Organize pipeline work | `stage('Stage Name') { ... }` |

---

## 9. Common Troubleshooting Scenarios

| Problem | What to check |
|---|---|
| Git checkout fails | Repository URL, branch name, Git plugin, credentials |
| `mvn: command not found` | Maven installation and `PATH` |
| Maven build fails | `pom.xml`, JDK version, dependencies, test failures |
| SonarQube analysis fails | Server configuration, token, network, scanner plugin |
| Nexus deployment fails | `distributionManagement`, repository permissions, server IDs and credentials |
| Tomcat deployment fails | WAR path, Manager application, credentials, network and HTTP response |
| Jenkins cannot find WAR file | Build output path, packaging type, artifact filename |
| Pipeline syntax error | Braces, parentheses, quotes and Jenkins Pipeline steps |

---

## 10. Useful Reference

- [Jenkins Scripted Pipeline Guide](https://www.jenkins.io/doc/book/pipeline/)
- [Jenkins Pipeline Getting Started](https://www.jenkins.io/doc/book/pipeline/getting-started/)
- [Your Jenkins Scripted Pipeline Tutorial](https://blog.kkdevops.com/jenkins-scripted-pipeline-a-complete-step-by-step-guide-for-real-ci-cd/)

**Interview tip:** Be ready to explain the purpose of `node`, `stage`, `sh`, `tool`, `withSonarQubeEnv`, `withCredentials`, `mvn clean package`, and `mvn deploy`. Explain what each stage does, how Jenkins passes artifacts between stages, and how you troubleshoot a failed build or deployment.
