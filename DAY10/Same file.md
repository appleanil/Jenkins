# Blog
```
https://blog.kkdevops.com/jenkins-ci-cd-upstream-downstream-triggers-parallel-execution-shared-libraries/
```
# Jenkins Shared Libraries – Complete Guide

## 1. What Is a Jenkins Shared Library?

A Jenkins Shared Library is a collection of reusable pipeline functions, Groovy classes, and resources that can be shared across multiple Jenkins pipelines to reduce code duplication and simplify maintenance.

## 2. Components of a Shared Library

- **`vars/`** – Contains reusable global pipeline functions that can be called directly from a Jenkinsfile.
- **`src/`** – Contains Groovy classes and reusable business logic.
- **`resources/`** – Stores non-Groovy files, such as JSON, YAML, and configuration files.

## 3. Example Shared Library Structure

```text
Sharedlib-kkfunda/
├── vars/
│   ├── myUtils.groovy
│   └── sendSlackNotifications.groovy
├── src/
│   └── com/
│       └── example/
│           └── MyClass.groovy
└── resources/
    └── config/
        └── myConfig.json
```

## 4. Create a Shared Library Function

### File: `vars/myUtils.groovy`

```groovy
def call(String name) {
    echo "Hello, ${name}!"
}
```

### How to Use It

```groovy
@Library('SharedLibKKFunda') _

pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                myUtils('Anil')
            }
        }
    }
}
```

**Output:** `Hello, Anil!`

## 5. Create a Groovy Class

### File: `src/com/example/MyClass.groovy`

```groovy
package com.example

class MyClass implements Serializable {
    def steps

    MyClass(steps) {
        this.steps = steps
    }

    void greet(String name) {
        steps.echo("Hello, ${name}!")
    }
}
```

### How to Use It

```groovy
@Library('SharedLibKKFunda') _

import com.example.MyClass

pipeline {
    agent any

    stages {
        stage('Greeting') {
            steps {
                script {
                    def obj = new MyClass(this)
                    obj.greet('Anil')
                }
            }
        }
    }
}
```

**Note:** Passing the pipeline steps object allows the class to call Jenkins pipeline steps such as `echo`.

## 6. Create a Resource File

### File: `resources/config/myConfig.json`

```json
{
    "key": "value",
    "environment": "development"
}
```

### How to Read the Resource

```groovy
@Library('SharedLibKKFunda') _

pipeline {
    agent any

    stages {
        stage('Read Configuration') {
            steps {
                script {
                    def config = libraryResource('config/myConfig.json')
                    echo config
                }
            }
        }
    }
}
```

**Note:** `libraryResource()` reads a file from the configured Shared Library's `resources/` directory.

---

## 7. How to Configure a Global Shared Library in Jenkins

### Step 1: Create a GitHub Repository

Create or use your shared library repository:

`https://github.com/kkdevopsb6/Sharedlib-kkfunda.git`

Add your `vars/`, `src/`, and `resources/` folders to the repository and push the files to GitHub.

### Step 2: Configure Jenkins

Navigate to:

**Jenkins Dashboard → Manage Jenkins → System → Global Pipeline Libraries**

Configure the following:

| Setting | Value |
|---|---|
| Name | `SharedLibKKFunda` |
| Default version | `main` |
| Retrieval method | Modern SCM |
| Source Code Management | Git |
| Project Repository | `https://github.com/kkdevopsb6/Sharedlib-kkfunda.git` |
| Credentials | Select a Jenkins credential if the repository is private |

Click **Save**.

**Important:** `SharedLibKKFunda` is the library name, while `main` is the default Git branch. They serve different purposes.

## 8. Access the Shared Library from a Jenkinsfile

Add this annotation at the beginning of your Jenkinsfile:

```groovy
@Library('SharedLibKKFunda') _
```

The library name must exactly match the **Name** configured under Global Pipeline Libraries.

## 9. Create a Reusable Slack Notification Function

### File: `vars/sendSlackNotifications.groovy`

```groovy
def call(String buildStatus = 'SUCCESS') {
    def colors = [
        STARTED: '#FFFF00',
        SUCCESS: '#00FF00',
        FAILURE: '#FF0000',
        ABORTED: '#808080'
    ]

    def colorCode = colors.get(buildStatus, '#FF0000')

    def message = "${buildStatus}: Job '${env.JOB_NAME} " +
                  "[${env.BUILD_NUMBER}]' - ${env.BUILD_URL}"

    slackSend(
        color: colorCode,
        message: message,
        channel: '#jio-devops'
    )
}
```

**Prerequisite:** Install and configure the Jenkins Slack Notification plugin, including valid Slack credentials and workspace settings.

## 10. Complete Jenkins Declarative Pipeline Using the Shared Library

This example uses the shared library for Slack notifications and keeps the Maven, SonarQube, Nexus, and Tomcat stages in the Jenkinsfile.

```groovy
@Library('SharedLibKKFunda') _

pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'development',
                    url: 'https://github.com/kkdevopsb5/maven-webapplication-project-kkfunda.git'

                script {
                    sendSlackNotifications('STARTED')
                }
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('SonarQube Report') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Deploy to Nexus') {
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

    post {
        success {
            script {
                sendSlackNotifications('SUCCESS')
            }
        }

        failure {
            script {
                sendSlackNotifications('FAILURE')
            }
        }

        aborted {
            script {
                sendSlackNotifications('ABORTED')
            }
        }
    }
}
```

### Important Configuration Notes

1. Replace `<TOMCAT-IP>` with your actual Tomcat server IP.
2. Create a Jenkins username/password credential with ID `tomcat-manager-credentials`.
3. Configure SonarQube authentication and Nexus publishing credentials separately.
4. The Maven `deploy` goal requires the correct Nexus repository configuration in `pom.xml` and Maven `settings.xml`.
5. The example assumes the GitHub repository is public. If it is private, configure a Jenkins Git credential.
6. For production, use HTTPS or another secure network path for Tomcat deployment.
7. If Slack notifications should not affect the build result, add suitable error handling around the shared library's `slackSend` call.

---

## 11. How to Import Multiple Shared Libraries

### Option 1: Import Multiple Libraries Separately

```groovy
@Library('SharedLibKKFunda') _
@Library('my-shared-lib-2') _
@Library('my-shared-lib-3') _

pipeline {
    agent any

    stages {
        stage('Example') {
            steps {
                echo 'Multiple shared libraries loaded'
            }
        }
    }
}
```

### Option 2: Import Multiple Libraries in One Statement

```groovy
@Library([
    'SharedLibKKFunda',
    'my-shared-lib-2@main',
    'my-shared-lib-3@develop'
]) _

pipeline {
    agent any

    stages {
        stage('Example') {
            steps {
                echo 'Multiple shared libraries loaded'
            }
        }
    }
}
```

**Note:** Each library must be configured in Jenkins, and the specified branch or version must exist. A version can be specified using `@branch`, `@tag`, or another supported Git reference.

---

## 12. Interview Questions and Single-Line Answers

1. **What is a Jenkins Shared Library?** A reusable collection of pipeline functions, Groovy classes, and resources shared across Jenkins pipelines.
2. **What is the purpose of `vars/`?** It contains global pipeline variables and reusable functions.
3. **What is the purpose of `src/`?** It contains reusable Groovy classes and supporting logic.
4. **What is the purpose of `resources/`?** It stores non-Groovy files such as JSON and YAML configuration files.
5. **What does `@Library('SharedLibKKFunda') _` do?** It loads the configured Shared Library into the Jenkins pipeline.
6. **Can multiple Jenkins jobs use the same Shared Library?** Yes, multiple jobs can reuse the same centrally maintained library.
7. **What is `call()` in a Shared Library?** It defines the function that allows a global variable script to be invoked like a pipeline step.
8. **What is the main benefit of Shared Libraries?** They reduce duplicated pipeline code and simplify centralized maintenance.
9. **Can Shared Libraries contain Slack notifications?** Yes, reusable Slack notification logic can be stored in a file such as `vars/sendSlackNotifications.groovy`.
10. **Can Shared Libraries use Git branches or tags?** Yes, a configured library can load a supported branch, tag, or other version reference.
