# 🚀Automated CI/CD Pipeline with Jenkins, Docker, and AWS ECR
This project demonstrates how to build an automated CI/CD pipeline using Jenkins to:

✅ Build a Docker image from your codebase

✅ Push the Docker image to Amazon Elastic Container Registry (ECR)

✅ Trigger the pipeline using GitHub Webhooks

✅ Works on Amazon Linux 2 EC2 instance
🐳 Powered by Docker, Jenkins, and AWS ECR

📁 Project Structure
```
bash
.
├── Dockerfile               # Sample Dockerfile used for image creation
├── Jenkinsfile              # Jenkins pipeline script
├── README.md                # Project documentation
├── app/                     # Example application source code
└── scripts/                 # Helper shell scripts (if needed)
```
🧩 Prerequisites
Before proceeding, make sure you have:

AWS Account with ECR access

EC2 instance (Amazon Linux 2) with:

Jenkins installed

Docker installed and running

Jenkins user added to Docker group

GitHub repository containing the code and Dockerfile

IAM role/policy configured to allow ECR push (e.g., AmazonEC2ContainerRegistryFullAccess)

🔧 Part 1 – Launch Jenkins Server on EC2 (Amazon Linux 2)
Launch EC2 Instance

Amazon Linux 2, t2.micro (free tier), open port 8080 for Jenkins

Install Jenkins & Docker
```
bash
sudo yum update -y
sudo yum install java-11-openjdk -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io-2023.key
sudo yum install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Install Docker
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker

# Add Jenkins to docker group
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
Access Jenkins

Visit http://<EC2-PUBLIC-IP>:8080

Unlock using initial password from:
```
bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Install Suggested Plugins and Setup Admin User

🏗️ Part 2 – Prepare GitHub Repo and AWS ECR
🔹 1. Create AWS ECR Repository
```
bash
aws ecr create-repository --repository-name my-app --region <your-region>
Note the repository URI:
<account-id>.dkr.ecr.<region>.amazonaws.com/my-app
```
🔹 2. Create GitHub Repository
Push your project code to GitHub.

Include Dockerfile, Jenkinsfile, and app source code.

🔹 3. Add GitHub Webhook
Navigate to Settings → Webhooks

Add:

Payload URL: http://<JENKINS-IP>:8080/github-webhook/

Content type: application/json

Events: Just push event

⚙️ Part 3 – Create Jenkins Pipeline
Go to Jenkins → New Item → Pipeline

Configure:

GitHub repository URL

Choose “Pipeline script from SCM” → Git

Save and build once to verify setup.

✅ Example Jenkinsfile Highlights
```
groovy
Copy
Edit
pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '<your-account-id>'
        AWS_REGION = 'us-east-1'
        IMAGE_REPO_NAME = 'my-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-username/your-repo.git'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .'
            }
        }
        stage('Docker Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                """
            }
        }
        stage('Docker Tag and Push') {
            steps {
                sh """
                docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
                """
            }
        }
    }
}
```
🧼 Part 4 – Clean Up ECR Resources
To delete the image repository (when no longer needed):
```
bash
aws ecr delete-repository --repository-name my-app --force --region <your-region>
```
📌 Notes
Make sure Jenkins has the proper IAM credentials (e.g., through EC2 instance role or AWS credentials)

If using private repo, configure GitHub credentials in Jenkins

Consider adding Slack notifications or deployment stages in future enhancements

📚 References
AWS ECR Docs

Jenkins Pipeline

Docker Basics

GitHub Webhooks
