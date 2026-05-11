# Jenkins_Lab_2
---
## Lab 2: Containerization & Code Quality – Docker, SonarQube, Private Repo & New Secrets
---

## Task 1 : Secure Access to Private Repository (using PAT)
Building on Lab 1, you will now move your application code to a Private GitHub Repository.
To enable Jenkins to pull this private code, you will use your Personal Access Token (PAT) as the authentication method.
Action: Convert the repository to Private and configure Jenkins to authenticate via PAT for secure code retrieval.
Steps:

1. Repository Visibility Update
Go to your GitHub repository Settings.

Scroll down to the Danger Zone.

Click Change visibility and set the repository to Private.

Note: After this change, Jenkins will fail to pull the code until the next steps are completed.

2. Configure Jenkins Credentials
In the Jenkins Dashboard, go to Manage Jenkins > Credentials.

Click on the (global) domain and select Add Credentials.

Kind: Select Username with password.

Username: Enter your GitHub username.

Password: Paste the Personal Access Token (PAT) you generated in Lab 1.

ID: Enter github-pat-creds (This ID will be used in your Jenkinsfile).

Click Create.

3. Update the Pipeline Script (Jenkinsfile)
Modify your Jenkinsfile to use the new credentials ID.
ex: 
stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: '<link of your repo>',
                        credentialsId: 'github-pat-creds'
                    ]]
                )
            }
}


---


## Task 2: Static Code Analysis Setup (SonarQube)
Before shipping the code, we must analyze its quality and find potential bugs or security vulnerabilities.

Step 1: Start the SonarQube Server
- you can start one instantly using Docker
- $ docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
- Access the dashboard at: http://<YOUR_SERVER_IP>:9000
- Default Credentials: Username: admin / Password: admin (You will be prompted to change the password immediately).

Step 2: Create a Project and Generate a Token
 - Jenkins needs a "key" (token) to upload the analysis results to SonarQube.
 - In the SonarQube dashboard, click Create Project -> Select Manually.
 - Project Key: Enter service-app
 - Click Set Up.
 - Under "How do you want to analyze your repository?", select With Jenkins.
 - Go to Jenkins Dashboard and install SonarQube Scanner plugin 
 - Follow the prompts to Generate a token:
     - Name the token: jenkins-sonar-token
     - Click Generate
     - Copy the token immediately. You will not be able to see it again!

Step 3: Store the Token in Jenkins Credentials
 - you must give this "key" to Jenkins so it can talk to SonarQube during the pipeline execution
 - Go to your Jenkins Dashboard > Manage Jenkins > Credentials
 - Click on the (global) domain and select Add Credentials.
 - Kind: Select Secret text.
 - Secret: Paste the token you copied from SonarQube.
 - ID: Enter sonar-token (This ID will be used in your Jenkinsfile).
 - Click Create.

**Phase 2: Jenkins Global Configuration**
You need to tell Jenkins where the SonarQube server is and which scanner tool to use.
Step 1: Install SonarQube Scanner Tool
Go to Manage Jenkins > Tools.

Scroll to SonarQube Scanner.

Click Add SonarQube Scanner.

Name: SonarScanner (This exact name must be used in your Jenkinsfile).

Check Install automatically.

Step 2: Configure SonarQube Server Settings
Go to Manage Jenkins > System.

Scroll to SonarQube servers.

Click Add SonarQube.

Name: SonarQube-Server (This exact name must be used in your Jenkinsfile).

Server URL: Use your VM IP (e.g., http://192.168.1.10:9000).

Note: Do not use localhost if Jenkins is in a Docker container.

Server authentication token: Add the token you copied earlier as a Secret Text credential in Jenkins with the ID sonar-token.

## Phase 3: Jenkinsfile Configuration
Add the analysis stage to your pipeline Jenkinsfile.

## Verify SonarQube
After sonarqube success, edit the code in src/OrderProcessor.php *Remove the comment hash*
then try to push again => should sonarqube failed