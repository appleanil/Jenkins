# Jenkins by KK FUNDA — Interview Notes and Installation Guide

## 1. Explain the CI/CD Flow Diagram

### Flow

```text
Developer
    |
    v
  GitHub
    |
    v
  Jenkins
    |
    v
   Maven
    |
    v
 SonarQube
    |
    v
   Nexus
    |
    v
  Tomcat
    |
    v
 Email Notification
```

### Step-by-step explanation

**1. Developer**

Developers write application code, fix bugs, and implement new features. Once the code is ready, they commit and push it to GitHub.

**2. GitHub — Source Code Management**

GitHub stores the application's source code and version history. Jenkins can start a pipeline when code changes are pushed, using a webhook or another configured trigger.

**3. Jenkins — CI/CD Automation**

Jenkins automates the software delivery process. It checks out the source code, runs build commands, coordinates testing and code analysis, publishes artifacts, deploys applications, and sends notifications.

**4. Maven — Build Automation**

Maven compiles the Java application, resolves dependencies, runs configured tests, and packages the application into a JAR or WAR file.

Example:

```bash
mvn clean package
```

**5. SonarQube — Code Quality Analysis**

SonarQube analyzes source code to identify code-quality issues, bugs, code smells, and security-related findings.

Example:

```bash
mvn sonar:sonar
```

Jenkins can enforce a configured quality gate before allowing the pipeline to continue.

**6. Nexus — Artifact Repository**

Nexus Repository Manager stores versioned build artifacts, such as JAR and WAR files. This allows teams to manage and retrieve approved artifacts for deployment.

Example:

```bash
mvn deploy
```

**7. Tomcat — Application Deployment**

Tomcat runs compatible Java web applications packaged as WAR files. Jenkins can deploy a WAR to Tomcat using its Manager API or another approved deployment mechanism.

**8. Email Notification**

Jenkins sends notifications to developers, QA teams, or other stakeholders about build and deployment results.

Examples:
- Build successful.
- Build failed.
- SonarQube quality gate failed.
- Deployment completed.

### Real-time example

Suppose a developer changes a Java application used by a company.

1. The developer pushes code to GitHub.
2. GitHub triggers Jenkins.
3. Jenkins checks out the latest code.
4. Maven builds the application and runs tests.
5. SonarQube analyzes the code.
6. Jenkins publishes the approved artifact to Nexus.
7. Jenkins deploys the artifact to Tomcat.
8. Jenkins sends an email notification.

**Important:** This is a common pipeline design. The actual order may vary. For example, a team can perform code analysis before packaging, and a quality-gate failure can stop the pipeline before publishing or deployment.

---

## 2. What Is Jenkins?

### Interview Question

What is Jenkins?

### Answer

Jenkins is an open-source automation server written in Java. It is widely used for Continuous Integration (CI) and Continuous Delivery or Continuous Deployment (CD).

Jenkins automates repetitive software development tasks, including:

- Source code checkout.
- Application compilation and build.
- Automated testing.
- Code-quality analysis.
- Artifact publishing.
- Application deployment.
- Notifications and reporting.

### History of Jenkins

- Jenkins originated as Hudson, a continuous integration project created by Kohsuke Kawaguchi.
- The project began in 2004.
- Following a community and project governance dispute, Jenkins emerged as a fork of Hudson in 2011.

### Example

```text
GitHub → Jenkins → Maven → SonarQube → Nexus → Tomcat
```

Jenkins coordinates these tools and automates the workflow.

### Interview Answer

Jenkins is an open-source automation server that helps implement CI/CD pipelines. In my DevOps workflow, Jenkins integrates with GitHub for source code, Maven for builds, SonarQube for code analysis, Nexus for artifact management, and Tomcat for application deployment.

---

## 3. Advantages of Jenkins

1. **Open source:** Jenkins is freely available under an open-source license.
2. **Large plugin ecosystem:** Plugins integrate Jenkins with many tools and platforms.
3. **Platform support:** Jenkins can run on supported operating systems and environments.
4. **Automation:** It reduces manual build, testing, and deployment tasks.
5. **CI/CD integration:** It supports automated software delivery pipelines.
6. **Extensibility:** Pipelines can be written using Jenkins Pipeline syntax and Groovy-based Scripted Pipelines.
7. **Reporting and notifications:** It can publish test reports and notify stakeholders.
8. **Community support:** Documentation, community discussions, and tutorials are widely available.

Plugin directory: https://plugins.jenkins.io/

---

## 4. What Is Continuous Integration (CI)?

### Definition

Continuous Integration is a software development practice in which developers frequently integrate code changes into a shared version-control repository. Each integration is verified through automated builds and tests to identify problems early.

### CI Flow Diagram

```text
Developer Writes Code
          |
          v
     Git Commit
          |
          v
     Push to GitHub
          |
          v
     Jenkins Trigger
          |
          v
     Checkout Code
          |
          v
      Maven Build
          |
          v
     Automated Tests
          |
          v
    SonarQube Analysis
          |
          v
     Build Feedback
```

The exact order of testing and code analysis depends on the pipeline configuration.

### CI Advantages

- Detects bugs early.
- Reduces integration problems.
- Provides faster feedback to developers.
- Improves consistency of builds.
- Encourages frequent integration of changes.
- Supports automated testing and code-quality checks.

### Example

A developer pushes code to GitHub. Jenkins automatically checks out the code, runs `mvn clean verify`, and reports whether the build and tests succeeded.

---

## 5. What Is Continuous Delivery?

### Definition

Continuous Delivery is a software development practice in which every change that passes the required automated build, testing, and quality checks can be released to production through a controlled process.

Production deployment may require a manual approval.

### Continuous Delivery Flow

```text
Code Changes
     |
     v
Unit Tests
  (Auto)
     |
     v
Build and Integration Tests
       (Auto)
     |
     v
Acceptance Tests
     (Auto)
     |
     v
Production Approval
     (Manual, if required)
     |
     v
Deploy to Production
```

### Example

For a client project, the pipeline builds the application, runs tests, performs quality checks, and deploys it to a staging environment. After QA approval and client sign-off, the application is released to production.

### Advantages

- Reduces release risk.
- Makes software releases more predictable.
- Allows teams to release when business requirements are met.
- Supports approval controls and release management.
- Ensures the application is ready for production deployment.

**Important:** Continuous Delivery does not always require manual approval. The defining point is that production release remains controlled and may require an explicit release decision.

---

## 6. What Is Continuous Deployment?

### Definition

Continuous Deployment is a practice in which every change that passes the required automated checks is automatically deployed to production without a manual production-release approval.

### Continuous Deployment Flow

```text
Code Changes
     |
     v
Unit Tests
  (Auto)
     |
     v
Build and Integration Tests
       (Auto)
     |
     v
Acceptance Tests
     (Auto)
     |
     v
Quality Gates
     (Auto)
     |
     v
Deploy to Production
     (Auto)
```

### Advantages

- Faster delivery of features and fixes.
- Reduced manual release effort.
- Faster feedback from production.
- Frequent and smaller releases.
- More automated and repeatable deployments.

Continuous Deployment requires strong automated testing, monitoring, rollback or recovery mechanisms, and suitable production safeguards.

---

## 7. What Is the Difference Between CI, Continuous Delivery, and Continuous Deployment?

| Feature | Continuous Integration | Continuous Delivery | Continuous Deployment |
|---|---|---|---|
| Main goal | Integrate and validate code frequently | Keep software ready for release | Automatically release every change that passes checks |
| Automated builds | Yes | Yes | Yes |
| Automated tests | Expected | Expected | Expected |
| Production deployment | Not required | Controlled; may require approval | Automatic |
| Manual production approval | Not applicable to CI itself | May be required | Not required for each release |
| Main focus | Integration and feedback | Release readiness | Fully automated production releases |

### Interview Question

Which approach are you using: Continuous Delivery or Continuous Deployment?

### Sample Answer

In projects where production releases require client approval, QA sign-off, or a change-management process, Continuous Delivery is suitable because the release remains controlled.

For internal projects where the organization permits automatic production releases and the automated tests and safeguards are mature, Continuous Deployment can be used.

The choice depends on the project's requirements, risk level, business policies, and production controls.

---

## 8. What Can Jenkins Do?

Jenkins can automate many software delivery activities.

- Integrate with GitHub, GitLab, Bitbucket, and other version-control systems.
- Build Java applications using Maven or Gradle.
- Run automated tests and publish JUnit test reports.
- Integrate with SonarQube for code-quality analysis.
- Publish artifacts to Nexus Repository Manager or JFrog Artifactory.
- Deploy applications to Tomcat, virtual machines, containers, and Kubernetes.
- Build and publish Docker images.
- Send email or messaging notifications.
- Execute infrastructure automation and deployment scripts.
- Run parameterized and scheduled jobs.

---

## 9. Popular Continuous Integration and CI/CD Tools

| No. | Tool | Open Source / Commercial |
|---:|---|---|
| 1 | Jenkins | Open source |
| 2 | CloudBees CI | Commercial enterprise offering |
| 3 | Bamboo | Commercial |
| 4 | CruiseControl | Open source |
| 5 | Travis CI | Hosted service with plan-dependent offerings |
| 6 | CircleCI | Hosted service with plan-dependent offerings |
| 7 | GitLab CI/CD | Available in GitLab's open-source and commercial offerings |
| 8 | TeamCity | Commercial product with plan-dependent offerings |
| 9 | GitHub Actions | GitHub-hosted automation with usage and plan-dependent limits |

### Interview Answer

Jenkins is a popular choice because it is open source, supports a large plugin ecosystem, and integrates with many build, testing, security, artifact, and deployment tools.

---

# 10. Jenkins Installation on AWS EC2

## Prerequisites

Before installing Jenkins, prepare:

- An AWS account with permission to launch EC2 instances.
- A supported Linux distribution.
- A compatible Java Development Kit (JDK).
- Network connectivity to the Jenkins package repository.
- Security Group rules for SSH and Jenkins web access.
- Sufficient CPU, RAM, and disk space for the expected workload.

### Instance type

A `t2.medium` has 2 vCPUs and 4 GiB of RAM and can be suitable for a small learning environment. Actual requirements depend on plugins, builds, and concurrent jobs.

For production, select an instance size based on workload and resource monitoring.

---

## 11. Step-by-Step Jenkins Installation on a RHEL-Compatible EC2 Instance

The following example uses a RHEL-compatible system and the Jenkins LTS repository. Package availability and commands may vary by Linux distribution and release.

### Step 1: Launch an EC2 Instance

1. Open the AWS Management Console.
2. Navigate to EC2.
3. Launch an instance using a supported Linux AMI.
4. Select an appropriate instance type.
5. Configure a key pair.
6. Configure the Security Group.
7. Launch the instance.

Connect through SSH using the appropriate username for the AMI.

Example:

```bash
ssh -i "your-key.pem" ec2-user@<EC2-PUBLIC-IP>
```

### Step 2: Switch to an Administrative Shell

```bash
sudo -i
```

Update package metadata and install basic utilities:

```bash
yum install wget tree -y
```

On distributions that use `dnf`, use the appropriate `dnf` commands if needed.

### Step 3: Install a Compatible Java Version

Jenkins requires a supported Java version. Confirm the Java requirements for the Jenkins LTS release you intend to install.

For a compatible RHEL-based system with Java 21 packages available:

```bash
yum install fontconfig java-21-openjdk -y
```

Verify Java:

```bash
java -version
```

If necessary, install the JDK package appropriate to the distribution.

### Step 4: Add the Jenkins LTS Repository

```bash
wget -O /etc/yum.repos.d/jenkins.repo \
  https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

Import the Jenkins repository signing key:

```bash
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

Update package metadata:

```bash
yum upgrade -y
```

Install Jenkins:

```bash
yum install jenkins -y
```

Reload systemd configuration:

```bash
systemctl daemon-reload
```

### Step 5: Enable and Start Jenkins

Enable Jenkins at boot:

```bash
systemctl enable jenkins
```

Start Jenkins:

```bash
systemctl start jenkins
```

Check its status:

```bash
systemctl status jenkins
```

If Jenkins fails to start, inspect its logs:

```bash
journalctl -u jenkins -n 100 --no-pager
```

### Step 6: Configure the AWS Security Group

Jenkins uses TCP port `8080` by default unless configured otherwise.

In the EC2 Security Group, add an inbound rule:

| Type | Protocol | Port | Source |
|---|---|---:|---|
| SSH | TCP | 22 | Your trusted public IP |
| Custom TCP | TCP | 8080 | Your trusted public IP or approved network |

Avoid opening Jenkins to the entire internet unless there is a specific, justified requirement and appropriate protections are in place.

### Step 7: Open Jenkins in the Browser

Navigate to:

```text
http://<EC2-PUBLIC-IP>:8080/
```

Replace the placeholder with the actual public IP address or DNS name of your EC2 instance.

For production use, prefer HTTPS behind an appropriately configured reverse proxy or load balancer.

### Step 8: Retrieve the Initial Administrator Password

Run:

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the generated password and enter it into the Jenkins setup page.

### Step 9: Complete the Initial Setup

1. Select **Install suggested plugins**.
2. Wait for the plugins to install.
3. Create your own administrator account.
4. Configure the Jenkins URL.
5. Click **Save and Finish**.
6. Click **Start using Jenkins**.

Use a unique, strong password or an approved identity provider. Do not reuse public tutorial credentials.

### Step 10: Verify Jenkins

Check service status:

```bash
systemctl status jenkins
```

Check the listening port:

```bash
ss -lntp | grep ':8080'
```

Check the HTTP endpoint locally:

```bash
curl -I http://localhost:8080/
```

A valid HTTP response indicates that Jenkins is responding locally. If the browser cannot connect, check the Security Group, host firewall, routing, service status, and logs.

---

# 12. Important Jenkins Installation Commands

| Task | Command |
|---|---|
| Switch to root | `sudo -i` |
| Check Java version | `java -version` |
| Install Java 21 on a compatible RHEL-based system | `yum install fontconfig java-21-openjdk -y` |
| Install Jenkins | `yum install jenkins -y` |
| Enable Jenkins at boot | `systemctl enable jenkins` |
| Start Jenkins | `systemctl start jenkins` |
| Stop Jenkins | `systemctl stop jenkins` |
| Restart Jenkins | `systemctl restart jenkins` |
| Check Jenkins status | `systemctl status jenkins` |
| View Jenkins logs | `journalctl -u jenkins -n 100 --no-pager` |
| Get initial administrator password | `cat /var/lib/jenkins/secrets/initialAdminPassword` |
| Check port 8080 | `ss -lntp \| grep ':8080'` |
| Test Jenkins locally | `curl -I http://localhost:8080/` |

---

# 13. Jenkins Interview Quick Revision

**Q1. What is Jenkins?**

Jenkins is an open-source automation server used to build, test, and deliver software through CI/CD pipelines.

**Q2. What is the default Jenkins port?**

TCP port `8080` is the default HTTP port unless it is changed.

**Q3. What is the difference between CI and CD?**

CI validates frequently integrated code changes through automated builds and tests. CD can mean Continuous Delivery, which keeps software ready for controlled release, or Continuous Deployment, which automatically releases changes that pass all required checks.

**Q4. What is a Jenkins plugin?**

A plugin extends Jenkins functionality or integrates it with other tools, such as Git, Maven, SonarQube, Nexus, and email services.

**Q5. What is a Jenkins Pipeline?**

A Jenkins Pipeline defines an automated workflow as code. It can contain stages for checkout, building, testing, analysis, artifact publishing, deployment, and notifications.

**Q6. What is a Jenkins agent?**

An agent is a machine or execution environment where Jenkins runs pipeline steps and jobs.

**Q7. What is a Jenkins workspace?**

A workspace is the directory allocated to a job or pipeline on an agent, commonly used for checked-out source code and build files.

**Q8. How does Jenkins integrate with GitHub?**

Jenkins checks out source code using the Git plugin and can trigger builds through webhooks, polling, or other configured triggers.

**Q9. Why do we use Nexus?**

Nexus stores and manages build artifacts and dependencies so teams can version, share, and retrieve approved software packages.

**Q10. Why do we use SonarQube?**

SonarQube analyzes code quality and security-related issues and can enforce configured quality gates in CI/CD pipelines.

**Q11. Why do we use Tomcat?**

Tomcat runs compatible Java web applications, commonly deployed as WAR files.

**Q12. How do you troubleshoot Jenkins if the UI is not opening?**

Check the service status, application logs, listening port, Security Group, firewall, network routing, and Jenkins URL.

---

# 14. End-to-End Interview Explanation

If the interviewer asks you to explain your Jenkins CI/CD workflow, you can answer:

"In our CI/CD workflow, developers push source code to GitHub. Jenkins retrieves the code and starts the pipeline. Maven compiles the application, runs automated tests, and generates the artifact. SonarQube analyzes code quality and the configured quality gate determines whether the pipeline can continue. Once approved, Maven publishes the versioned artifact to Nexus. Jenkins deploys the approved artifact to Tomcat and sends an email notification with the build and deployment status.

This automation reduces manual effort, provides faster feedback, improves consistency, and makes software releases more reliable."

---

## Official References

- Jenkins documentation: https://www.jenkins.io/doc/
- Jenkins installation documentation: https://www.jenkins.io/doc/book/installing/
- Jenkins plugins: https://plugins.jenkins.io/
- Jenkins Pipeline documentation: https://www.jenkins.io/doc/book/pipeline/
- Jenkins Red Hat package repository: https://pkg.jenkins.io/redhat-stable/

