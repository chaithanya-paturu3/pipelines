# pipelines
ci/cd
Skanda Apartments CI/CD Pipeline Flow
1. Project Setup

Created a GitHub repository pipelines.

Added the Skanda Apartments landing page HTML (index.html) and style.css.

Initial landing page features:

Welcome header

Modern Design, Prime Location, 24/7 Security sections

Footer with copyright info

2. EC2 Instance Setup

Launched a free-tier Ubuntu EC2 instance on AWS.

Installed Docker and Jenkins:

# Update packages
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker

# Install Jenkins
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins


Confirmed disk space usage with:

df -h


Ensured enough free space to run Jenkins and Docker containers.

3. Docker & Jenkins Integration

Docker images are used to build and deploy the app.

Jenkins pipelines can use Docker containers as agents or run directly on the host.

Previous Docker image: node:18-alpine for building Node.js-based content (for flexibility if using Node scripts).

4. Jenkins Pipeline Setup
Pipeline File (Jenkinsfile)
pipeline {
    agent any   // Use host node with Docker installed

    environment {
        DOCKERHUB_USER = 'chaithu33'
        IMAGE_NAME = 'skanda-apartments'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKERHUB_USER/$IMAGE_NAME:latest skanda-landing/'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                        docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker stop skanda || true
                    docker rm skanda || true
                    docker run -d -p 8081:80 --name skanda $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        success { echo '🎉 Skanda Landing Page deployed successfully!' }
        failure { echo '❌ Deployment failed!' }
    }
}

5. Jenkins Credentials

Created Docker Hub access token.

Added it in Jenkins:

Dashboard → Manage Jenkins → Credentials → System → Global credentials → Add Credentials

Kind: Secret text

Secret: Docker Hub token

ID: dockerhub-pass (must match Jenkinsfile)

6. GitHub Repository & Webhooks

Pushed initial code to GitHub.

Enabled webhook in GitHub to trigger Jenkins automatically on every push:

Repository → Settings → Webhooks → Add webhook

Payload URL: http://<JENKINS_IP>:8080/github-webhook/

Content type: application/json

Select: Just the push event

Checked “GitHub hook trigger for GITScm polling” in Jenkins pipeline job.

7. CI/CD Flow

Code change: Update index.html or style.css (e.g., add address and owners info).

Push code to GitHub → Webhook triggers Jenkins.

Jenkins pipeline stages:

Build Docker Image → Build latest version of app in Docker

Push Docker Image → Upload to Docker Hub

Deploy Container → Stop old container, run new container on port 8081

Access landing page:

http://<EC2_PUBLIC_IP>:8081


Every subsequent push triggers the pipeline automatically.
