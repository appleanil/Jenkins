మీ Declarative Pipeline notes‌ను సరిచేసేటప్పుడు ముఖ్యంగా ఈ విషయాలు గుర్తుంచుకోండి:

- `pipeline` spelling సరిగా ఉండాలి; `pipepline` తప్పు.
- Maven tool name Jenkins‌లో configure చేసిన పేరుతో సరిగ్గా match కావాలి.
- Approval stage‌ను deployment‌కు ముందు ఉంచాలి; సాధారణంగా Nexus upload తర్వాత, Tomcat deployment‌కు ముందు ఉంచుతారు.
- `post { success { ... } failure { ... } }` ద్వారా build పూర్తయ్యాక Slack notification పంపవచ్చు.
- Tomcat password‌ను pipeline‌లో నేరుగా రాయకుండా Jenkins Credentials ఉపయోగించాలి.

# Jenkins Declarative Pipeline – Corrected Notes

## 1. Basic Declarative Pipeline Syntax

```
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello Jenkins'
            } // End of steps
        } // End of stage
    } // End of stages
} // End of pipeline
```

## 2. Git Checkout

```
pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'anil-test',
                    url: 'https://github.com/appleanil/maven-webapplication-project-kkfunda.git'
            }
        }
    }
}
```

## 3. Maven Build

```
pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
    }

    stages {
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

## 4. SonarQube Report

```
pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
    }

    stages {
        stage('SonarQube Report') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }
    }
}
```

**Note:** Configure the SonarQube server, authentication, and Maven plugin before running this stage.

## 5. Upload Artifact to Nexus

```
pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
    }

    stages {
        stage('Upload to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }
    }
}
```

**Note:** Configure `distributionManagement` in `pom.xml` and Nexus credentials in Maven `settings.xml`.

## 6. Deploy to Tomcat

```
pipeline {
    agent any

    stages {
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
                        "http://<TOMCAT-IP>:8080/manager/text/deploy?path=/maven-web-application&update=true"
                    '''
                }
            }
        }
    }
}
```

**Note:** Replace `<TOMCAT-IP>` with your Tomcat server IP and configure the Jenkins credential ID. Ensure the Tomcat Manager application permits deployment from Jenkins.

## 7. Slack Notifications

Configure the Slack Notification plugin under **Manage Jenkins → System → Slack** and configure the required Slack credentials.

```
post {
    success {
        script {
            notifyBuild('SUCCESS')
        }
    }

    failure {
        script {
            notifyBuild('FAILURE')
        }
    }

    aborted {
        script {
            notifyBuild('ABORTED')
        }
    }
}
```

### Slack Notification Function

```
def notifyBuild(String buildStatus = 'STARTED') {
    def colorCode

    switch (buildStatus) {
        case 'STARTED':
            colorCode = '#FFFF00'
            break
        case 'SUCCESS':
            colorCode = '#00FF00'
            break
        case 'ABORTED':
            colorCode = '#808080'
            break
        default:
            colorCode = '#FF0000'
    }

    def message = "${buildStatus}: Job '${env.JOB_NAME} " +
                  "[${env.BUILD_NUMBER}]' - ${env.BUILD_URL}"

    slackSend(
        color: colorCode,
        message: message,
        channel: '#jio-devops'
    )
}
```

To send a start notification, call `notifyBuild('STARTED')` inside the first stage.

## 8. Manual Approval Stage

Place the approval stage after the checks and artifact upload stages, immediately before deployment.

```
stage('Approval') {
    steps {
        script {
            slackSend(
                channel: '#jio-devops',
                message: "Build awaiting approval: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )

            timeout(time: 10, unit: 'MINUTES') {
                input(
                    message: 'Approve deployment?',
                    ok: 'Approve'
                )
            }
        }
    }
}
```

**Note:** The example waits up to 10 minutes for approval. If nobody approves within that period, the pipeline times out.

## 9. Complete Declarative Pipeline

```
pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
    }

    stages {
        stage('Git Checkout') {
            steps {
                script {
                    notifyBuild('STARTED')
                }

                git branch: 'development',
                    url: 'https://github.com/kkdevopsb6/maven-webapplication-project-kkfunda.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Report') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }

        stage('Approval') {
            steps {
                script {
                    slackSend(
                        channel: '#jio-devops',
                        message: "Deployment approval required for ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                    )

                    timeout(time: 10, unit: 'MINUTES') {
                        input(
                            message: 'Approve deployment to Tomcat?',
                            ok: 'Approve'
                        )
                    }
                }
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
                        "http://<TOMCAT-IP>:8080/manager/text/deploy?path=/maven-web-application&update=true"
                    '''
                }
            }
        }
    }

    post {
        success {
            script {
                notifyBuild('SUCCESS')
            }
        }

        failure {
            script {
                notifyBuild('FAILURE')
            }
        }

        aborted {
            script {
                notifyBuild('ABORTED')
            }
        }
    }
}

// Slack notification function
def notifyBuild(String buildStatus = 'STARTED') {
    def colors = [
        STARTED: '#FFFF00',
        SUCCESS: '#00FF00',
        FAILURE: '#FF0000',
        ABORTED: '#808080'
    ]

    slackSend(
        color: colors.get(buildStatus, '#FF0000'),
        channel: '#jio-devops',
        message: "${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${env.BUILD_URL}"
    )
}
```

**Pipeline flow:** Git Checkout → Build → SonarQube → Nexus → Manual Approval → Tomcat Deployment → Slack Notification.

**Important:** This example assumes the Maven, SonarQube, Nexus, Tomcat, and Slack configurations already exist. If you want to block deployment when SonarQube quality checks fail, configure and enforce a SonarQube Quality Gate before the approval stage.

Consolidate everything into one runnable pipelineAdd Jenkins prerequisites and setup steps
