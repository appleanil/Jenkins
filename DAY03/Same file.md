# Jenkins – Important Concepts and Plugins

## 1. SCM Polling
Jenkins periodically checks the source code repository and triggers a build when new changes are detected.

**Pros:** Automatically detects code changes.  
**Cons:** Frequent polling consumes resources.

## 2. Periodic Builds
Jenkins runs builds according to a cron schedule, regardless of whether code changes exist.

**Pros:** Regular automated builds.  
**Cons:** May rebuild unchanged code.

## 3. GitHub Webhook
GitHub notifies Jenkins when repository events occur, allowing builds to start without waiting for polling.

**Pros:** Fast build triggering with less polling.  
**Cons:** Requires correct webhook and network configuration.

### GitHub Webhook Configuration
1. Jenkins job → **Configure → Build Triggers → GitHub hook trigger for GITScm polling**.
2. GitHub repository → **Settings → Webhooks → Add webhook**.
3. Payload URL: `http://<JENKINS-IP>:8080/github-webhook/`
4. Content type: `application/json`.
5. Select the required events, usually **Just the push event**, and click **Add webhook**.
6. Ensure GitHub can reach Jenkins; use HTTPS and appropriate security controls for production.

## 4. Code Coverage – JaCoCo
JaCoCo measures Java code coverage and generates reports showing which code is tested.

**Goal:** Maintain at least 80% coverage.  
**Note:** Configure a coverage threshold check in the build so that coverage below 80% fails the build.

## 5. Discard Old Builds
Configure **Job → Configure → Discard old builds** to remove old build records based on retention limits.

## 6. Copy an Existing Jenkins Job
Select **New Item → Copy from**, enter the existing job name, choose the job type if required, and click **OK**.

## 7. Jenkins Server IP Change
Update the Jenkins URL under **Manage Jenkins → System → Jenkins Location** if the server IP changes.

**Note:** Do not manually edit `jenkins.model.jenconf.xml`; this is not the standard Jenkins configuration file for the Jenkins URL. Check the actual server configuration and restart only if necessary.

## 8. Deploy to Container Plugin
Deploys application artifacts, such as WAR files, to supported application servers such as Apache Tomcat.

## 9. Maven Integration Plugin
Integrates Maven with Jenkins to build, test, and package Java applications.

## 10. Restart vs. Safe Restart

- **Restart:** Restarts Jenkins and may interrupt running jobs.
- **Safe Restart:** Waits for running jobs to finish before restarting Jenkins.

**Linux command:** `sudo systemctl restart jenkins`

**Web endpoints:** `http://<JENKINS-URL>/restart` and `http://<JENKINS-URL>/safeRestart`, where supported and authorized.

**Note:** Restart endpoints require appropriate permissions and may be restricted by Jenkins security settings. Safe restart functionality may be available through Jenkins core or plugins, depending on the installation.

## 11. Next Build Number Plugin
Allows you to set the next build number for a job; it must be greater than the previous build number.

**Note:** The job's build-number file is normally located at `/var/lib/jenkins/jobs/<JOB-NAME>/nextBuildNumber` on a standard Linux installation, but paths vary with the Jenkins home directory and folder structure.

## 12. SSH Agent Plugin
Allows Jenkins jobs and pipelines to use configured SSH credentials to connect securely to remote servers.

## 13. Audit Trail Plugin
Records Jenkins activities, such as job creation, deletion, and configuration changes.

### Audit Trail Configuration
1. Install the Audit Trail plugin.
2. Go to **Manage Jenkins → System → Audit Trail**.
3. Add a logger and configure the log file path.
4. Example path: `/var/lib/jenkins/audit-trail.log`
5. Configure log rotation, for example, 20 MB per file and 5 retained files.

**View logs:** `sudo tail -f /var/lib/jenkins/audit-trail.log`

**Note:** Rotated filenames depend on the logging configuration. Ensure the Jenkins service account can write to the log directory.

## 14. Blue Ocean Plugin
Provides a visual interface for viewing and troubleshooting Jenkins pipelines.

## 15. Build Name and Description Setter Plugin
Allows custom build names and descriptions.

**Example build name:** `jio-dev-#${BUILD_NUMBER}`

**Configuration:** Job → Configure → Build Environment → Set Build Name, if supported by the installed plugin version.

## 16. ThinBackup Plugin
Backs up selected Jenkins configuration and job data to support recovery.

## 17. Role-Based Authorization
Assigns permissions to Jenkins users according to their roles, such as Admin, Developer, and Tester.

**Note:** Configure an appropriate authorization strategy and grant only the permissions each role requires.

## 18. Build with Parameters
Allows users to select inputs, such as branch names or environments, before starting a build.

### Choice Parameter Configuration
1. Job → Configure → General.
2. Select **This project is parameterized**.
3. Add a **Choice Parameter**.
4. Name: `BranchNames`
5. Choices:
   - `main`
   - `dev`
   - `qa`
6. Save the job.

**Branch specification:** `*/${BranchNames}`

**Note:** This works when the SCM configuration supports the selected branch specification and the branch exists in the repository.

## 19. Create a Jenkins View
Click **New View** on the Jenkins dashboard, enter a name, choose a view type, select the jobs to display, and click **Create** or **OK**.

## 20. Enable or Disable a Jenkins Job
- **Enable:** Open a disabled job and select **Enable** to allow builds again.
- **Disable:** Open a job and select **Disable Project** to prevent new builds from starting.

