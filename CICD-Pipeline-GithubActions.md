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
*   **AWS Setup:** AWS VPC, EC2 Instance, IAM User with ECR Full Access,ECR Repo, SNS Topic and Subscription.

Note: 
* Below Github Scretes needs to be created and feed with the proper valid values.
* EC2 Instance Prerequisite to Install Docker and AWS CLI.
 
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

## System Architecture
<img width="777" height="853" alt="image" src="https://github.com/user-attachments/assets/d0b57f12-1e9a-41fb-82d5-8f8d6adac328" />
---

## 🚀 Pipeline Workflow & Implementation Steps

Our GitHub Actions workflow is broken down into 8 distinct stages. Below is a detailed explanation of what happens at each stage and what you should expect to see in the execution logs.

1. Create a VPC Network.
<img width="1603" height="720" alt="image" src="https://github.com/user-attachments/assets/584100c9-5fea-4e85-96fc-d044647a48bc" />

2. Create a **EC2-ECR-Access-Role** IAM Policy with below access and Inbound Rules assigned.
<img width="1888" height="858" alt="image" src="https://github.com/user-attachments/assets/d6c5023b-9d08-4442-9ae7-0470060e2303" />
<img width="1608" height="707" alt="image" src="https://github.com/user-attachments/assets/7969356a-a2c2-42a9-bdea-627d0095cc55" />


3. Create IAM user assign **CLI-ECR-PUSH-PULL-ACCESS-POLICY** access.
<img width="1913" height="803" alt="image" src="https://github.com/user-attachments/assets/81efb648-e131-411d-ace2-a0378d818389" />
<img width="1902" height="802" alt="image" src="https://github.com/user-attachments/assets/a56446f4-6c74-4ff8-ac65-f3d4ab19b5e7" />

## 🔐 IAM Role Permissions
AWS CLI user access role to push and pull ECR Repo.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "GetAuthorizationToken",
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken"
            ],
            "Resource": "*"
        },
        {
            "Sid": "ManageRepositoryContents",
            "Effect": "Allow",
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:DescribeImages",
                "ecr:BatchGetImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload",
                "ecr:PutImage"
            ],
            "Resource": "arn:aws:ecr:us-east-1:071146695294:repository/flaskcicdpipeline"
        }
    ]
}
```

4. Create EC2 Instance and assign **EC2-ECR-Access-Role** also Install Docker and AWS CLI.
<img width="1600" height="750" alt="image" src="https://github.com/user-attachments/assets/9d846548-6f76-4285-ac97-070976b51a72" />

5. ECR Repo
<img width="1903" height="862" alt="image" src="https://github.com/user-attachments/assets/3dbf5681-c8d5-48da-b580-021a5e966cbc" />


6. Create SNS Topic and Subscriber.
<img width="1907" height="782" alt="image" src="https://github.com/user-attachments/assets/8be85450-f01b-4ea6-a817-eae4be5555b3" />
<img width="1897" height="793" alt="image" src="https://github.com/user-attachments/assets/1dfc066b-bca6-431b-9488-b28cd383845d" />

7. Confirm the SNS Service Subscription.
<img width="1915" height="962" alt="image" src="https://github.com/user-attachments/assets/b2b0a13a-55b7-4220-9dc9-b80a27923c20" />

8. Setup Github Secrets Variables and configure with proper values.
<img width="1906" height="958" alt="image" src="https://github.com/user-attachments/assets/40f3c325-ae6e-41a5-8e27-4f377fbe7e02" />

9. Execuete autodeployment.yaml workflow.

10. Execute Successful and Failure Screnario and check whether Alerts receiving receiving or not.
<img width="1892" height="867" alt="image" src="https://github.com/user-attachments/assets/2ca7c817-21e3-492f-969d-f66dad94d56c" />
<img width="1885" height="867" alt="image" src="https://github.com/user-attachments/assets/99f7a55b-ed6a-4007-91c5-c1f809d905ec" />

11. Alert Notification 
<img width="1565" height="757" alt="image" src="https://github.com/user-attachments/assets/fd3f642e-d236-4d57-8a5a-dc55474c2204" />
<img width="1561" height="762" alt="image" src="https://github.com/user-attachments/assets/32757371-bcd4-41e5-92a7-d90f767a7cbe" />

