# Jenkins CI/CD Practical Guide — KK FUNDA

## 1. Objective

Install and integrate the following DevOps tools with Jenkins and automate a Java web application's build, code analysis, artifact publishing, deployment, and email notification.

- GitHub — Source Code Management
- Maven — Build Automation
- SonarQube — Code Quality Analysis
- Nexus Repository — Artifact Management
- Apache Tomcat — Application Deployment
- Jenkins — CI/CD Automation
- Email — Build and Deployment Notifications

## 2. Complete CI/CD Architecture

```text
Developer
    |
    v
 GitHub Repository
    |
    v
 Jenkins Freestyle Job
    |
    v
 Maven: Clean + Package + Tests
    |
    v
 SonarQube: Code Analysis
    |
    v
 Nexus: Publish Artifact
    |
    v
 Tomcat: Deploy WAR
    |
    v
 Email Notification
```

### Tool installation links

| Tool | GitHub repository |
|---|---|
| SonarQube | https://github.com/SonarSource/docker-sonarqube |
| Nexus Repository | https://github.com/sonatype/nexus-public |
| Nexus installer | https://github.com/sonatype-nexus-community/nexus-repository-installer |
| Apache Tomcat | https://github.com/apache/tomcat |
| Jenkins | https://github.com/jenkinsci/jenkins |

Use the repositories' README files for installation instructions. Choose compatible Java, operating system, and product versions. For production deployments, follow the product's official installation and security documentation as well.

---

# 3. Install Git on the Jenkins Server

Git is required for Jenkins to check out the application source code.

### RHEL / Amazon Linux

```bash
sudo yum install git -y
git --version
```

On distributions that use `dnf`:

```bash
sudo dnf install git -y
git --version
```

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install git -y
git --version
```

If Jenkins uses a separate build agent, ensure Git is installed on the agent that executes the checkout.

---

# 4. Create a Jenkins Freestyle Job

### Step 1: Create the job

1. Open the Jenkins Dashboard.
2. Click **New Item**.
3. Enter the item name: `jio-dev`.
4. Select **Freestyle project**.
5. Click **OK**.

### Step 2: Configure the description

Under **General**, enter:

```text
Java web application CI/CD job using GitHub, Maven,
SonarQube, Nexus Repository and Tomcat.
```

### Step 3: Configure GitHub

Under **Source Code Management**:

1. Select **Git**.
2. Enter the repository URL:

```text
https://github.com/appleanil/maven-webapplication-project-kkfunda.git
```

3. Configure credentials if the repository is private.
4. Under **Branches to build**, enter:

```text
*/development
```

This assumes the repository has a branch named `development`. The branch name must match the actual GitHub branch.

5. Save the configuration.

### Step 4: Verify the checkout

Click **Build Now**.

Open the job and select **Console Output**.

If checkout fails, check:

- Git installation.
- Repository URL.
- Branch name.
- Jenkins Git plugin.
- Repository credentials and network access.

---

# 5. Configure Maven in Jenkins

## Why configure Maven in Jenkins?

Jenkins can use a configured Maven installation to execute builds. You do not have to install and maintain a separate manually configured Maven installation for every job.

Jenkins-managed tool installations may download Maven to a directory on the controller or build agent, depending on the configuration. Maven can also be installed separately on an agent and configured by its installation path.

### Step 1: Configure Maven

1. Open **Manage Jenkins**.
2. Select **Tools**.
3. Find **Maven installations**.
4. Click **Add Maven**.
5. Set the name to:

```text
maven-3.9.9
```

6. Select the desired Maven version.
7. Enable automatic installation if required.
8. Save the configuration.

### Step 2: Add Maven to the Freestyle job

1. Open `jio-dev`.
2. Click **Configure**.
3. Scroll to **Build Steps**.
4. Click **Add build step**.
5. Select **Invoke top-level Maven targets**.
6. Select `maven-3.9.9` as the Maven version.
7. Enter the goals:

```text
clean package
```

8. Save the configuration.
9. Click **Build Now**.

### Step 3: Verify the build artifact

The project may generate a WAR file under:

```text
/var/lib/jenkins/workspace/jio-dev/target/
```

Example:

```text
/var/lib/jenkins/workspace/jio-dev/target/maven-web-application.war
```

The actual filename depends on the project's `pom.xml`.

### Common build failure

If Maven is not configured correctly, check the console output and verify the selected Maven installation. You can also inspect the configured Maven version using:

```bash
mvn -version
```

This command works only if Maven is available in the shell's `PATH`.

---

# 6. Integrate SonarQube with Jenkins

## Objective

Analyze application code and display quality results in SonarQube.

### Step 1: Install SonarQube

GitHub repository:

https://github.com/SonarSource/docker-sonarqube

Follow its README for the Docker-based installation. If installing directly on a Linux server, use the appropriate SonarQube installation documentation instead.

For a typical standalone SonarQube web interface, the default URL is:

```text
http://<SONARQUBE-IP>:9000/
```

Replace the placeholder with the actual server address.

### Step 2: Install the Jenkins plugin

1. Open **Manage Jenkins**.
2. Select **Plugins**.
3. Search for **SonarQube Scanner**.
4. Install the plugin.
5. Restart Jenkins if the installation process requires it.

### Step 3: Configure the SonarQube server

1. Open **Manage Jenkins → System**.
2. Find **SonarQube servers**.
3. Add a server configuration.
4. Set the name to:

```text
SonarQubeServer
```

5. Enter the actual SonarQube server URL.
6. Select a Jenkins credential containing the SonarQube token.
7. Save the configuration.

Do not store the SonarQube authentication token in the public Git repository.

### Step 4: Configure Maven analysis

For a Maven project, the analysis can be triggered using the SonarQube Maven goal.

Example Maven goals:

```text
clean package sonar:sonar
```

The SonarQube Maven plugin must be available and compatible with the project. Depending on the plugin and server configuration, you may need to configure the project key, server URL, authentication, and Java version.

A server URL can be configured in `pom.xml` as a non-secret property, or supplied through the build configuration. Keep credentials in Jenkins credentials.

### Step 5: Run the job

1. Open `jio-dev`.
2. Click **Configure**.
3. Update the Maven build goals to include the SonarQube analysis.
4. Save the job.
5. Click **Build Now**.
6. Open **Console Output**.
7. Open SonarQube and verify the project analysis.

### Best practice

Configure a SonarQube quality gate and make the pipeline fail when the required quality conditions are not met. A successful analysis alone does not necessarily mean the project passed the quality gate.

---

# 7. Integrate Nexus Repository with Jenkins

## Objective

Publish versioned JAR or WAR artifacts to Nexus so that they can be stored and retrieved for deployment.

### Step 1: Install Nexus Repository

GitHub repositories:

- https://github.com/sonatype/nexus-public
- https://github.com/sonatype-nexus-community/nexus-repository-installer

Follow the selected repository's instructions and confirm that the Nexus version, installation method, and Java requirements are compatible.

### Step 2: Create Maven repositories

In the Nexus interface, create or verify the required Maven repositories.

For this example, the repository names are:

```text
jio-release
jio-snapshot
```

The release repository stores release versions, while the snapshot repository stores development snapshot versions.

Repository names and URLs must match your Nexus configuration.

### Step 3: Configure `distributionManagement` in `pom.xml`

Add the following inside the root `<project>` element:

```xml
<distributionManagement>
    <repository>
        <id>nexus</id>
        <name>KK FUNDA Releases Nexus Repository</name>
        <url>http://<NEXUS-HOST>:8081/repository/jio-release/</url>
    </repository>

    <snapshotRepository>
        <id>nexus</id>
        <name>KK FUNDA Snapshot Nexus Repository</name>
        <url>http://<NEXUS-HOST>:8081/repository/jio-snapshot/</url>
    </snapshotRepository>
</distributionManagement>
```

Replace `<NEXUS-HOST>` with your actual Nexus hostname or IP address.

**Important:** Use HTTPS for repository access wherever possible, particularly when credentials are transmitted.

### Step 4: Configure Nexus credentials

Maven uses `settings.xml` to store server authentication details.

A common settings file for a Maven installation managed by Jenkins might be located under the Maven installation's `conf/` directory. However, a better practice is to use a dedicated Maven settings file with controlled permissions and inject it into the build through Jenkins credentials or an approved configuration mechanism.

Example server configuration:

```xml
<settings>
    <servers>
        <server>
            <id>nexus</id>
            <username>YOUR_NEXUS_USERNAME</username>
            <password>YOUR_NEXUS_PASSWORD</password>
        </server>
    </servers>
</settings>
```

Replace the placeholders with credentials supplied securely. Do not commit this file with real passwords to GitHub.

The server `<id>` must match the repository `<id>` in `distributionManagement`.

### Step 5: Locate Maven settings files

To find settings files on Linux:

```bash
find / -name settings.xml 2>/dev/null
```

This can return several results, including global and user settings. Verify which settings file your Jenkins build actually uses.

### Step 6: Configure the Maven deployment goal

In the Freestyle job:

1. Open `jio-dev`.
2. Click **Configure**.
3. Under **Build Steps**, configure the Maven invocation.
4. Ensure the Nexus credentials and settings file are available.
5. Set the goals to:

```text
clean deploy
```

If you also need SonarQube analysis, configure the analysis step and quality-gate handling separately.

### Step 7: Verify the Nexus upload

Click **Build Now**.

Open the Console Output and look for a successful Maven deployment. Then open Nexus and verify that the expected artifact and version appear in the appropriate repository.

**Important:** Maven's `deploy` phase publishes artifacts to a Maven repository; it does not deploy the application to Tomcat.

---

# 8. Integrate Tomcat with Jenkins

## Objective

Deploy the generated WAR file to Tomcat after the build and quality checks succeed.

### Step 1: Install Tomcat

GitHub repository:

https://github.com/apache/tomcat

Use the repository's README and the official Tomcat documentation to choose the correct release and installation method.

Confirm that the Tomcat server is running and reachable from the Jenkins agent.

### Step 2: Install the Jenkins plugin

1. Open **Manage Jenkins → Plugins**.
2. Search for **Deploy to container**.
3. Install the plugin.
4. Restart Jenkins if required.

### Step 3: Configure Tomcat credentials

Create a Jenkins username/password credential for an authorized Tomcat Manager account.

Configure Tomcat with the required Manager application and permissions. Do not expose Manager access unnecessarily or reuse insecure example passwords.

### Step 4: Configure the deployment action

1. Open the `jio-dev` job.
2. Click **Configure**.
3. Find **Post-build Actions**.
4. Select **Deploy war/ear to a container**.
5. Set the WAR file path relative to the workspace:

```text
**/maven-web-application.war
```

6. Under **Containers**, select the Tomcat version compatible with your deployment plugin.
7. Enter the Tomcat URL, for example:

```text
http://<TOMCAT-HOST>:8080/
```

8. Select the configured Tomcat credentials.
9. Save the job.

The plugin's supported Tomcat versions and connection requirements depend on its installed version.

### Step 5: Run the job

1. Click **Build Now**.
2. Open **Console Output**.
3. Verify the deployment result.
4. Open the application URL:

```text
http://<TOMCAT-HOST>:8080/maven-web-application/
```

The context path depends on the deployed WAR name or the deployment configuration.

---

# 9. Configure Email Notifications

## Objective

Notify stakeholders when the build or deployment succeeds or fails.

### Step 1: Configure SMTP

1. Open **Manage Jenkins → System**.
2. Configure the email extension or mail settings.
3. Enter the SMTP server and port.
4. Configure authentication and TLS according to your email provider.
5. Store credentials securely.
6. Send a test email if the configured plugin supports it.

### Step 2: Configure the job

In the Freestyle job:

1. Click **Configure**.
2. Find **Post-build Actions**.
3. Add an email notification or email-ext action.
4. Configure recipients.
5. Select the required triggers, such as failure or success.
6. Save the job.

### Example notification

```text
Subject: Jenkins Build Status - jio-dev

Job: jio-dev
Build Number: <BUILD_NUMBER>
Status: <BUILD_STATUS>

Please review the Jenkins Console Output for details.
```

The placeholders above describe the intended notification content; configure the corresponding Jenkins variables or plugin tokens when building the actual email template.

---

# 10. Automate the Entire Process

## Recommended Freestyle Job Configuration

For a Freestyle project, a maintainable setup is to use distinct build steps and post-build actions, or to use a Jenkins Pipeline when more advanced stage control is required.

| Order | Stage / Action | Tool | Purpose |
|---:|---|---|---|
| 1 | Source Code Management | Git | Check out `development` |
| 2 | Build and Test | Maven | `clean package` |
| 3 | Code Analysis | SonarQube | Analyze source code and enforce the quality gate |
| 4 | Publish Artifact | Nexus | `deploy` to the configured Maven repository |
| 5 | Application Deployment | Tomcat | Deploy the WAR file |
| 6 | Notification | Email | Notify stakeholders of the result |

### Execution sequence

```text
Git Checkout
     |
     v
Maven Build and Tests
     |
     v
SonarQube Analysis
     |
     v
Quality Gate
     |
     v
Nexus Artifact Publishing
     |
     v
Tomcat Deployment
     |
     v
Email Notification
```

Configure the job so that a failed required build, test, analysis, or publishing step prevents later deployment steps from running.

---

# 11. Common Troubleshooting

| Problem | Troubleshooting |
|---|---|
| Git checkout fails | Check Git installation, repository URL, branch and credentials |
| Maven command fails | Check the configured Maven version, JDK, `pom.xml` and dependencies |
| SonarQube analysis fails | Check server URL, token, scanner configuration and network access |
| SonarQube project is missing | Check the project key, analysis logs and selected server |
| Nexus deployment fails | Check repository URL, artifact version, settings file, credentials and repository permissions |
| Tomcat deployment fails | Check WAR path, Tomcat Manager access, credentials, version compatibility and network rules |
| Email is not received | Check SMTP settings, credentials, TLS and Jenkins email logs |
| Build succeeds but application returns 404 | Check deployment status, context path and Tomcat logs |

---

# 12. Final Interview Explanation

"In my project, I use Jenkins to automate the CI/CD process. The job checks out source code from the development branch in GitHub, builds and tests the application using Maven, and analyzes the code through SonarQube. If the required quality gate passes, Maven publishes the versioned artifact to Nexus. Jenkins then deploys the WAR file to Tomcat and sends email notifications with the build and deployment status.

I configure the required tool installations and credentials in Jenkins, verify each stage through Console Output, and troubleshoot failures using build logs, repository configuration, network connectivity, and application logs."

