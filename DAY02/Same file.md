# Jenkins CI/CD Practical Guide — KK FUNDA

## 1. Objective

Install and integrate the following DevOps tools and automate a Java web application's build, analysis, artifact publishing, deployment, and email notification.

- GitHub — Source Code Management
- Maven — Build Automation
- SonarQube — Code Quality Analysis
- Nexus Repository — Artifact Management
- Apache Tomcat — Application Deployment
- Jenkins — CI/CD Automation
- Email — Notifications

## 2. Complete CI/CD Architecture

```text
Developer
    |
    v
 GitHub Repository
    |
    v
 Jenkins
    |
    v
 Maven Build + Tests
    |
    v
 SonarQube Analysis
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

---

# 3. Install Git on the Jenkins Server

For Amazon Linux 2023 or a compatible RHEL-based distribution:

```bash
sudo dnf install git -y
git --version
```

For Amazon Linux 2:

```bash
sudo yum install git -y
git --version
```

Git must be installed on the Jenkins agent that performs the checkout.

---

# 4. Install Jenkins on AWS EC2

### Step 1: Launch EC2

Launch an Amazon Linux 2023 instance, connect through SSH, and run:

```bash
sudo dnf update -y
sudo dnf install fontconfig java-21-openjdk wget git -y
java -version
```

Confirm that Java 21 is supported by the Jenkins release you install.

### Step 2: Configure the Jenkins repository

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
  https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import \
  https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

### Step 3: Install Jenkins

```bash
sudo dnf install jenkins -y
sudo systemctl daemon-reload
sudo systemctl enable --now jenkins
```

### Step 4: Verify Jenkins

```bash
sudo systemctl status jenkins
sudo journalctl -u jenkins -n 50 --no-pager
```

### Step 5: Configure the Security Group

Allow TCP port `8080` from your trusted IP address.

Open Jenkins:

```text
http://<JENKINS-PUBLIC-IP>:8080/
```

Retrieve the initial administrator password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Complete the browser setup, install suggested plugins, and create your own secure administrator account.

---

# 5. Install Maven in Jenkins

### Recommended method: Jenkins-managed Maven

1. Open **Manage Jenkins → Tools**.
2. Find **Maven installations**.
3. Click **Add Maven**.
4. Enter the installation name:

```text
maven-3.9.9
```

5. Select Maven 3.9.9 and enable automatic installation if required.
6. Save the configuration.

Verify the installation in a job or pipeline using:

```bash
mvn -version
```

If using Jenkins-managed tools, use the Maven installation configured in Jenkins rather than assuming that `mvn` is available globally.

---

# 6. Install SonarQube Using Docker

The following example uses Docker on an Amazon Linux 2023 EC2 instance. SonarQube requires sufficient memory and appropriate Linux kernel settings.

### Step 1: Install Docker

```bash
sudo dnf install docker -y
sudo systemctl enable --now docker
sudo docker --version
```

### Step 2: Configure the kernel setting

SonarQube's Elasticsearch component requires appropriate virtual-memory settings.

```bash
sudo sysctl -w vm.max_map_count=524288
```

To persist this setting across reboots:

```bash
echo 'vm.max_map_count=524288' |
  sudo tee /etc/sysctl.d/99-sonarqube.conf

sudo sysctl --system
```

Check the setting:

```bash
sysctl vm.max_map_count
```

### Step 3: Start SonarQube

```bash
sudo docker volume create sonarqube_data
sudo docker volume create sonarqube_extensions
sudo docker volume create sonarqube_logs
```

Run the Community Build image:

```bash
sudo docker run -d \
  --name sonarqube \
  --restart unless-stopped \
  -p 9000:9000 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  -v sonarqube_logs:/opt/sonarqube/logs \
  sonarqube:lts-community
```

### Step 4: Check the container

```bash
sudo docker ps
sudo docker logs -f sonarqube
```

Open:

```text
http://<SONARQUBE-PUBLIC-IP>:9000/
```

Sign in using the initial credentials specified by the product's setup instructions and change the default password.

**Important:** Allow TCP port `9000` only from trusted networks. For production, configure HTTPS and persistent storage, and review the memory, database, backup, and security requirements before deployment.

---

# 7. Install Nexus Repository Using Docker

This example uses the Nexus Repository 3 container image. Verify the current licensing, image availability, and supported deployment instructions before using it in production.

### Step 1: Create persistent storage

```bash
sudo docker volume create nexus_data
```

### Step 2: Run Nexus Repository

```bash
sudo docker run -d \
  --name nexus \
  --restart unless-stopped \
  -p 8081:8081 \
  -v nexus_data:/nexus-data \
  sonatype/nexus3
```

### Step 3: Verify the container

```bash
sudo docker ps
sudo docker logs -f nexus
```

### Step 4: Retrieve the initial administrator password

```bash
sudo docker exec nexus \
  cat /nexus-data/admin.password
```

If the file is no longer present, the initial setup may already have been completed.

### Step 5: Open Nexus

```text
http://<NEXUS-PUBLIC-IP>:8081/
```

Sign in using the initial password and complete the setup wizard.

### Step 6: Create Maven repositories

In the Nexus interface, create or verify these repositories:

- `jio-release` — Maven hosted release repository.
- `jio-snapshot` — Maven hosted snapshot repository.

Allow TCP port `8081` only from trusted networks. Use HTTPS and secure credentials when configuring production access.

---

# 8. Install Apache Tomcat Using Docker

This example uses Tomcat 9, which is appropriate only for applications compatible with its Servlet/JSP specifications. Choose the Tomcat version that matches your application's requirements.

### Step 1: Create persistent application storage

```bash
sudo docker volume create tomcat_webapps
```

### Step 2: Start Tomcat

Jenkins commonly uses port `8080`. To avoid a conflict when Tomcat runs on the same Docker host, expose Tomcat on host port `9090`:

```bash
sudo docker run -d \
  --name tomcat \
  --restart unless-stopped \
  -p 9090:8080 \
  -v tomcat_webapps:/usr/local/tomcat/webapps \
  tomcat:9.0-jdk21-temurin
```

### Step 3: Verify Tomcat

```bash
sudo docker ps
sudo docker logs -f tomcat
```

Test the HTTP endpoint:

```bash
curl -I http://localhost:9090/
```

Open:

```text
http://<TOMCAT-PUBLIC-IP>:9090/
```

**Important:** A Tomcat image may not include the Manager application by default. For deployment through the Manager API, install and configure the appropriate Manager application, access restrictions, and authorized credentials. Do not expose the Manager interface to the public internet.

For production, use an appropriately secured deployment process and persistent storage.

---

# 9. Create a Jenkins Freestyle Job

## Step 1: Create the job

1. Open Jenkins Dashboard.
2. Click **New Item**.
3. Enter the job name:

```text
jio-dev
```

4. Select **Freestyle project**.
5. Click **OK**.

## Step 2: Add a description

Under **General**, enter:

```text
Java web application CI/CD job using GitHub, Maven,
SonarQube, Nexus Repository and Tomcat.
```

## Step 3: Configure GitHub

Under **Source Code Management**:

1. Select **Git**.
2. Enter the repository URL:

```text
https://github.com/appleanil/maven-webapplication-project-kkfunda.git
```

3. Add credentials if the repository is private.
4. Under **Branches to build**, enter:

```text
*/development
```

The branch must exist in your GitHub repository.

5. Save the configuration.

## Step 4: Configure Maven build

Under **Build Steps**:

1. Click **Add build step**.
2. Select **Invoke top-level Maven targets**.
3. Select the configured Maven version.
4. Enter:

```text
clean package
```

5. Save the configuration and click **Build Now**.

## Step 5: Verify the WAR file

The workspace is commonly located at:

```text
/var/lib/jenkins/workspace/jio-dev/
```

Check the artifact directory:

```bash
ls -lh /var/lib/jenkins/workspace/jio-dev/target/
```

Example artifact:

```text
maven-web-application.war
```

The exact filename depends on the project's `pom.xml`.

---

# 10. Integrate SonarQube with Jenkins

### Step 1: Install the Jenkins plugin

Open **Manage Jenkins → Plugins**, search for **SonarQube Scanner**, and install it.

### Step 2: Configure the server

Open **Manage Jenkins → System** and add a SonarQube server.

Example configuration:

```text
Name: SonarQubeServer
URL: http://<SONARQUBE-HOST>:9000
```

Select a Jenkins credential containing a SonarQube token.

### Step 3: Run analysis

For a Maven project, configure the Maven goal:

```text
clean package sonar:sonar
```

The SonarQube Maven plugin must be available, and the project key, server configuration, authentication, and Java compatibility must be correct.

### Step 4: Verify

1. Save the Jenkins job.
2. Click **Build Now**.
3. Review **Console Output**.
4. Open SonarQube and verify the analysis result.

For a production pipeline, configure and enforce the quality gate before artifact publication or deployment.

---

# 11. Integrate Nexus Repository with Jenkins

## Step 1: Configure `pom.xml`

Add this inside the root `<project>` element:

```xml
<distributionManagement>
    <repository>
        <id>nexus</id>
        <name>KK FUNDA Releases</name>
        <url>http://&lt;NEXUS-HOST&gt;:8081/repository/jio-release/</url>
    </repository>

    <snapshotRepository>
        <id>nexus</id>
        <name>KK FUNDA Snapshots</name>
        <url>http://&lt;NEXUS-HOST&gt;:8081/repository/jio-snapshot/</url>
    </snapshotRepository>
</distributionManagement>
```

Replace the placeholder with the actual Nexus hostname or IP address.

Use HTTPS for secure repository access whenever possible.

## Step 2: Configure Maven credentials

Maven's `settings.xml` should contain a matching server ID.

Example:

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

Replace the placeholders with securely managed credentials. Never commit real passwords to GitHub.

Locate existing Maven settings files:

```bash
find / -name settings.xml 2>/dev/null
```

Verify which file is used by the Jenkins build. A dedicated settings file supplied through Jenkins credentials is preferable to editing a shared Maven installation's global configuration.

## Step 3: Publish the artifact

Configure the Maven deployment goal:

```text
clean deploy
```

Run the job and inspect Console Output.

Verify the artifact in Nexus under the appropriate release or snapshot repository.

**Note:** Maven `deploy` publishes artifacts to a Maven repository. It does not deploy the application to Tomcat.

---

# 12. Integrate Tomcat with Jenkins

## Step 1: Install the deployment plugin

Open **Manage Jenkins → Plugins** and install **Deploy to container**.

## Step 2: Configure the job

Open `jio-dev` and click **Configure**.

Under **Post-build Actions**:

1. Select **Deploy war/ear to a container**.
2. Enter the WAR file pattern:

```text
**/maven-web-application.war
```

3. Select a Tomcat version compatible with the installed plugin.
4. Configure the Tomcat URL:

```text
http://<TOMCAT-HOST>:9090/
```

5. Select the Jenkins credential for an authorized Tomcat Manager account.
6. Save the job.

## Step 3: Deploy and verify

Click **Build Now** and inspect Console Output.

Test the deployed application:

```text
http://<TOMCAT-HOST>:9090/maven-web-application/
```

The context path depends on the deployment configuration and application name.

---

# 13. Configure Email Notifications

## Step 1: Configure SMTP

1. Open **Manage Jenkins → System**.
2. Configure the mail server.
3. Enter the SMTP host and port.
4. Configure authentication and TLS.
5. Store credentials securely.
6. Test email delivery if supported.

## Step 2: Configure the job

Under the Freestyle job's **Post-build Actions**, add the required email notification action.

Configure recipients and success/failure triggers.

Example message:

```text
Subject: Jenkins Build Status - jio-dev

Job: jio-dev
Build Number: <BUILD_NUMBER>
Status: <BUILD_STATUS>

Review Jenkins Console Output for details.
```

Configure the plugin's supported variables or tokens to populate the actual build number and status.

---

# 14. Automate the Entire CI/CD Workflow

Recommended sequence:

| Order | Stage | Tool | Purpose |
|---:|---|---|---|
| 1 | Source checkout | GitHub | Retrieve application code |
| 2 | Build and tests | Maven | Compile, test, package |
| 3 | Code analysis | SonarQube | Analyze quality and security findings |
| 4 | Quality gate | SonarQube/Jenkins | Stop if required checks fail |
| 5 | Artifact publishing | Nexus | Store the versioned artifact |
| 6 | Application deployment | Tomcat | Deploy the approved WAR |
| 7 | Notification | Email | Notify stakeholders |

```text
GitHub
   |
   v
Jenkins Checkout
   |
   v
Maven Build + Tests
   |
   v
SonarQube Analysis
   |
   v
Quality Gate
   |
   v
Nexus Publishing
   |
   v
Tomcat Deployment
   |
   v
Email Notification
```

A Freestyle job can automate these steps through build steps and post-build actions. For complex workflows with explicit stages, approvals, quality-gate enforcement, and artifact promotion, a Jenkins Pipeline is often easier to maintain.

---

# 15. Troubleshooting Reference

| Issue | What to check |
|---|---|
| Git checkout failure | Git installation, repository URL, branch, credentials |
| Maven build failure | JDK, Maven version, dependencies, tests, `pom.xml` |
| SonarQube failure | Server URL, token, scanner configuration, network |
| Nexus upload failure | Repository URL, server ID, credentials, repository permissions |
| Tomcat deployment failure | WAR path, Manager access, credentials, port, network |
| Jenkins UI unavailable | Service status, port 8080, firewall, Security Group |
| Email not received | SMTP settings, authentication, TLS, notification triggers |

## Final Interview Answer

"In my CI/CD workflow, developers push code to GitHub, and Jenkins checks out the selected branch. Maven builds the Java application, runs automated tests, and generates a WAR file. SonarQube analyzes the source code and the quality gate controls whether the pipeline proceeds. Maven publishes the approved artifact to Nexus, and Jenkins deploys it to Tomcat. Finally, Jenkins sends email notifications with the build and deployment status.

I configure the required tools and credentials in Jenkins, verify each stage through Console Output, and troubleshoot issues using the relevant build logs, repository configuration, network connectivity, and application logs."

