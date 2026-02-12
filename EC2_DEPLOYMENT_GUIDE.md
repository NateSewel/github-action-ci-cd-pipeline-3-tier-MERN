# AWS EC2 & GitHub Actions Deployment Guide

This guide provides a step-by-step walkthrough to set up your AWS EC2 instance and configure GitHub Actions for automated CI/CD of the MERN application.

## 1. Prepare your EC2 Instance

Connect to your EC2 instance via SSH and run the following script to install all necessary dependencies (Docker, Docker Compose).

### Installation Script
Copy and paste this entire block into your EC2 terminal:

```bash
# Update system packages
sudo apt-get update && sudo apt-get upgrade -y

# Install Docker
sudo apt-get install -y docker.io

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Install Docker Compose (V2)
sudo apt-get install -y docker-compose-v2

# Add current user to docker group (Removes need for sudo)
sudo usermod -aG docker $USER

echo "Installation complete. PLEASE LOG OUT AND LOG BACK IN for group changes to take effect."
```

**Important**: After the script finishes, type `exit` to disconnect, then reconnect to your instance.

---

## 2. Configure GitHub Secrets

Navigate to your repository on GitHub: **Settings > Secrets and variables > Actions**. Add the following secrets:

| Secret Name | Value Description |
| :--- | :--- |
| **DOCKERHUB_USERNAME** | Your Docker Hub username. |
| **DOCKERHUB_TOKEN** | Your Docker Hub Access Token (Security > Access Tokens). |
| **EC2_HOST** | The Public IPv4 address or DNS of your EC2 instance. |
| **EC2_USERNAME** | Usually `ubuntu` (for Ubuntu) or `ec2-user` (for Amazon Linux). |
| **EC2_SSH_KEY** | The **entire contents** of your `.pem` private key file. |
| **SLACK_WEBHOOK** | The Webhook URL from your Slack App integration. |

---

## 3. SSH Key Authorization (Server Side)

For GitHub Actions to log into your server, your Public Key must be in the `authorized_keys` file.

1.  **Generate Public Key from your Private Key** (On your local machine):
    ```bash
    ssh-keygen -y -f your-key.pem
    ```
2.  **Add to EC2**:
    ```bash
    # On the EC2 instance
    nano ~/.ssh/authorized_keys
    ```
    Paste the output from the previous step on a new line and save (`Ctrl+O`, `Enter`, `Ctrl+X`).

---

## 4. GitHub Environments (Optional but Recommended)

Since the pipeline uses Environments for Staging and Production:
1.  Go to **Settings > Environments**.
2.  Create an environment named `production`.
3.  Create an environment named `staging`.
4.  You can add **Deployment Protection Rules** (like required reviewers) here.

---

## 5. Deployment Workflow

### Triggering a Deploy
-   **Staging**: Push or merge code into the `develop` branch.
-   **Production**: Push or merge code into the `main` branch.

### Manual Rollback
If a deployment fails or contains bugs:
1.  Go to the **Actions** tab in GitHub.
2.  Select the **Rollback Deployment** workflow on the left.
3.  Click **Run workflow**.
4.  Enter the **Docker Tag** (e.g., `20260209-abc1234`) and select the **Environment**.

---

## 6. Verification
Once the GitHub Action completes successfully, you can verify the containers on your EC2:
```bash
docker ps
```
Your frontend should be accessible at `http://your-ec2-ip:80`.
