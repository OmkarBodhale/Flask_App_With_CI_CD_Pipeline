# CI/CD Pipeline Implementation Guide

Welcome to the repository! This user manual is designed to help you quickly understand and implement our automated Continuous Integration and Continuous Deployment (CI/CD) pipeline using **GitHub Actions**. 

By following this guide and referencing the provided screenshots, you will be able to set up a robust pipeline that automatically builds, tests, and deploys the application to an AWS EC2 instance.

---

## 🛠️ Prerequisites & Setup

Before the pipeline can run successfully, ensure you have configured the necessary secrets in your GitHub repository (**Settings > Secrets and variables > Actions**):
*   **AWS Credentials:** Access Key ID and Secret Access Key for ECR and EC2 access.
*   **ECR Registry URI:** The URL to your target Elastic Container Registry.
*   **EC2 SSH Details:** Host IP, username, and SSH private key.
*   **Email Configuration:** AWS SNS Topic and Subscriber.

Note: Below Github Scretes needs to be created and feed with the proper valid values.
 
| COMPONENTS | GITHUB ACTIONS SECRETS | GITHUB ACTIONS SECRETS DESCRIPTION |
| :--- | :--- | :--- |
| **AWS Credentials** | `AWS_ACCESS_KEY_ID` | AWS SECRET ACCESS KEY ID |
| **AWS Credentials** | `AWS_SECRET_ACCESS_KEY` | AWS SECRET ACCESS KEY |
| **AWS Credentials** | `AWS_DEFAULT_REGION` | AWS REGION |
| **AWS Credentials** | `AWS_ACCOUNT_ID` | AWS ACCOUNT NUMBER |
| **AWS ECR Repo Name** | `ECR_REPO` | AWS ECR Repo Name |
| **EC2 Configuration** | `EC2_HOST` | EC2 PUBLIC IP ADDRESS |
| **EC2 Configuration**| `EC2_USERNAME` | EC2 USERNAME |
| **EC2 Configuration**| `EC2_SSH_KEY` | EC2 SSH_KEY |
| **EC2 Configuration**| `EC2_HOST_PORT` | EC2 HOST_PORT |
| **AWS SNS Topic Name** | `WORKFLOW_STATUS_SNS_TOPIC_NAME`| SNS TOPIC NAME |

---

## 🚀 Pipeline Workflow & Implementation Steps

Our GitHub Actions workflow is broken down into 8 distinct stages. Below is a detailed explanation of what happens at each stage and what you should expect to see in the execution logs.

### 1. Checkout
The pipeline initiates automatically on a push to the `main` branch. This stage uses the standard `actions/checkout` tool to pull the latest application source code into the GitHub Actions runner environment.
> **Expected Output:**
> ![Checkout Stage](screenshots/01_checkout.png)
> *The logs will show a successful fetch of the repository contents and commit history.*

### 2. Install
Once the code is available, the pipeline installs all required Python dependencies by executing `pip install -r requirements.txt`.
> **Expected Output:**
> ![Install Stage](screenshots/02_install.png)
> *The logs will display the downloading and installation of packages like Flask, Pytest, etc.*

### 3. Test
To ensure code quality and stability, `pytest` is executed against our test suite. **Note:** The pipeline is configured as a strict gate here; it will halt immediately if any single test fails.
> **Expected Output:**
> ![Test Stage](screenshots/03_test.png)
> *A summary showing all tests passing (e.g., `10 passed in 0.45s`).*

### 4. Build
With tests passed, the pipeline builds a new Docker image of the application. To ensure traceability, this image is uniquely tagged using the specific GitHub commit SHA (`${{ github.sha }}`).
> **Expected Output:**
> ![Build Stage](screenshots/04_build.png)
> *The Docker build output showing the layers being successfully packaged and tagged.*

### 5. Push to ECR
The runner authenticates with AWS and pushes the newly built and tagged Docker image to your designated Elastic Container Registry (ECR).
> **Expected Output:**
> ![Push to ECR](screenshots/05_push_ecr.png)
> *Logs confirming the image layers have been successfully pushed to the AWS registry URI.*

### 6. Deploy to EC2
The pipeline establishes a secure SSH connection to the target EC2 instance. It executes commands to:
1. Pull the new Docker image from ECR.
2. Stop and remove the old running container.
3. Start a new container using the freshly pulled image.
> **Expected Output:**
> ![Deploy Stage](screenshots/06_deploy_ec2.png)
> *SSH execution logs showing the Docker CLI commands executing successfully on the remote server.*

### 7. Verify
Deployment isn't considered complete until the app is confirmed alive. This stage acts as the final success gate by polling the application's `/health` endpoint.
> **Expected Output:**
> ![Verify Stage](screenshots/07_verify.png)
> *An HTTP 200 OK response received from the health check script/curl command.*

### 8. Notify
Regardless of whether the pipeline succeeded or failed at an earlier step, an email notification is dispatched. This ensures the development team is immediately aware of the deployment status.
> **Expected Output:**
> ![Notify Stage](screenshots/08_notify.png)
> *Log confirming the email was sent, alongside a screenshot of the actual email received in the inbox.*

---
*For further troubleshooting, check the **Actions** tab in this GitHub repository to view raw logs for any failed workflow runs.*
