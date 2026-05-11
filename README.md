# Lab 2: Containerization & Code Quality
> Docker · SonarQube · Private Repo · New Secrets

---

## Task 1 — Secure Access to Private Repository (PAT)

Move your app code to a private GitHub repo and configure Jenkins to authenticate via Personal Access Token (PAT).

---

### Step 1 — Make the repository private

1. Go to your GitHub repo **Settings**
2. Scroll down to **Danger Zone**
3. Click **Change visibility** → set to **Private**

> [!WARNING]
> Jenkins will fail to pull code until steps 2 and 3 are completed.

---

### Step 2 — Add credentials in Jenkins

Navigate to **Manage Jenkins → Credentials → (global) → Add Credentials** and fill in:

| Field | Value |
|---|---|
| Kind | `Username with password` |
| Username | Your GitHub username |
| Password | Your PAT from Lab 1 |
| ID | `github-pat-creds` |

Click **Create**.

---

### Step 3 — Update your Jenkinsfile

Add the credentials ID to the checkout stage:

```groovy
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
```

---

## Task 2 — Static Code Analysis with SonarQube

Analyse code quality and find bugs or security vulnerabilities before shipping.

---

### Phase 1 — SonarQube Server

#### Step 1 — Start SonarQube via Docker

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```

Then open the dashboard at `http://<YOUR_SERVER_IP>:9000`

> [!NOTE]
> Default credentials: **admin / admin** — you will be prompted to change the password on first login.

---

#### Step 2 — Create a project and generate a token

1. In the SonarQube dashboard: **Create Project → Manually**
2. Set project key to `service-app` → click **Set Up**
3. Under "How do you want to analyze?", select **With Jenkins**
4. Go to Jenkins and install the **SonarQube Scanner** plugin
5. Back in SonarQube, generate a token:

| Field | Value |
|---|---|
| Token name | `jenkins-sonar-token` |

> [!IMPORTANT]
> Copy the token immediately — it will not be shown again.

---

#### Step 3 — Store the token in Jenkins

Navigate to **Manage Jenkins → Credentials → (global) → Add Credentials**:

| Field | Value |
|---|---|
| Kind | `Secret text` |
| Secret | Paste your SonarQube token |
| ID | `sonar-token` |

Click **Create**.

---

### Phase 2 — Jenkins Global Configuration

#### Step 4 — Install SonarQube Scanner tool

Go to **Manage Jenkins → Tools → SonarQube Scanner → Add SonarQube Scanner**:

| Field | Value |
|---|---|
| Name | `SonarScanner` |
| Install automatically | ✅ checked |

> [!NOTE]
> The name `SonarScanner` must match exactly what is referenced in your Jenkinsfile.

---

#### Step 5 — Configure SonarQube server settings

Go to **Manage Jenkins → System → SonarQube servers → Add SonarQube**:

| Field | Value |
|---|---|
| Name | `SonarQube-Server` |
| Server URL | `http://<YOUR_VM_IP>:9000` |
| Server authentication token | `sonar-token` |

> [!WARNING]
> Do not use `localhost` as the server URL if Jenkins is running inside a Docker container.

---

### Phase 3 — Jenkinsfile Configuration

#### Step 6 — Add the analysis stage to your pipeline

Add the SonarQube stage to your existing Jenkinsfile using:
- Tool name: `SonarScanner`
- Server name: `SonarQube-Server`

---

### ✅ Verify SonarQube

After a successful SonarQube run:

1. Edit `src/OrderProcessor.php` — remove the comment hash (`#`)
2. Push the change
3. SonarQube should now **fail**, confirming quality gates are active