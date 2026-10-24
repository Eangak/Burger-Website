**Burger Website — Jenkins CI/CD, Docker Swarm & AWS**

**📖 Project Overview**

The Burger Website is a web application deployed using containerization and cloud technologies. This project demonstrates how GitHub, Jenkins, Docker, Docker Swarm, Amazon Web Services (AWS), Amazon Elastic Container Registry (Amazon ECR), and Cloudflare can work together in a DevOps workflow.

GitHub stores the application source code. Jenkins automates the build and deployment process. Docker packages the application into a container image, Amazon ECR stores the image, and Docker Swarm deploys and manages the application across AWS EC2 instances.

Cloudflare Tunnel can provide secure remote access to the Jenkins dashboard without requiring Jenkins to be directly exposed to the public internet.


**🎯 Project Objectives**

Host and maintain the Burger Website source code on GitHub.

Automate application builds and deployment using Jenkins.

Build and tag Docker images.

Push container images to Amazon ECR.

Deploy and manage the application using Docker Swarm.

Run multiple application replicas across Swarm nodes.

Provide remote access to Jenkins through Cloudflare Tunnel with appropriate access controls.

Practice deployment verification, version management, and rollback procedures.


**🛠️ Technologies Used**

Technology

Purpose

GitHub

Source-code hosting and version control

Jenkins

CI/CD pipeline automation

Docker

Containerization

Docker Swarm

Container orchestration and replica management

AWS EC2

Cloud infrastructure

Amazon ECR

Private Docker image registry

Cloudflare Tunnel

Remote access to Jenkins

Nginx

Web server, if configured in the application container


**🏗️ System Architecture**

Developer
   |
   v
GitHub Repository
   |
   v
Jenkins CI/CD Pipeline
   |
   v
Build Docker Image
   |
   v
Amazon ECR
   |
   v
AWS EC2 - Docker Swarm Manager
   |
   +----------------------+
   |                      |
   v                      v
Swarm Worker 1        Swarm Worker 2
   |                      |
   +----------+-----------+
              |
              v
       Burger Website
        HTTP Port 88

Jenkins Remote Access

Administrator's Browser
          |
          v
Cloudflare Access / Tunnel
          |
          v
Local Jenkins Server
      Port 8080

The Jenkins access flow assumes that Cloudflare Tunnel is configured to route requests to the Jenkins server and that suitable authentication and access policies are enabled.


**🔄 CI/CD Workflow**

The intended deployment workflow is:

Developer updates the website source code in GitHub.

Jenkins retrieves the source code.

Jenkins builds the Docker image.

Jenkins authenticates to Amazon ECR.

Jenkins pushes the versioned image to ECR.

Docker Swarm pulls the image and updates the service.

Jenkins or the administrator verifies the deployment.

A known-good image version can be redeployed when rollback is required.


**🐳 Docker Swarm Deployment**

The Burger Website runs as a Docker Swarm service.

The previously configured image and port mapping are:

ECR repository: burger-website

AWS Region: us-east-1

Image version: v1.1.0

Published port: 88

Container port: 80

Example image:

254209146027.dkr.ecr.us-east-1.amazonaws.com/burger-website:v1.1.0

Check the Swarm deployment with:

docker node ls
docker service ls
docker service ps burger-website_burger-website

The service name may differ if the stack or service has been renamed.


**☁️ AWS Amazon ECR**

Amazon ECR is used as the private container image registry.

The image repository follows this naming format:

ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/burger-website:TAG

Authenticate to ECR using AWS credentials or an appropriate IAM role. Avoid storing AWS access keys directly in source code or committing credentials to GitHub.


**🌐 Cloudflare Tunnel for Jenkins**

Cloudflare Tunnel can provide a controlled way to access Jenkins remotely.

Typical configuration:

Install cloudflared on the machine that can reach Jenkins.

Configure a tunnel to forward traffic to the Jenkins service, commonly http://localhost:8080.

Configure a hostname or temporary tunnel URL.

Apply Cloudflare Access policies to restrict access to authorized users.

Keep Jenkins authentication enabled and use HTTPS for external access.

Security note: A temporary Quick Tunnel URL is not a permanent production endpoint. For ongoing access, use a named tunnel and suitable access policies. Do not expose Jenkins publicly without authentication and appropriate protections.


**📁 Project Structure**

The repository may contain files similar to:

burger-website/
├── index.html
├── assets/
├── Dockerfile
├── docker-stack.yml
├── Jenkinsfile
└── README.md

Update this structure to match the actual files in the GitHub repository.


**🔍 Monitoring and Troubleshooting**

Useful Docker Swarm commands:

docker service ls
docker service ps burger-website_burger-website
docker service logs --tail 50 burger-website_burger-website
docker stats --no-stream


** 📌Project Status**

Application: Burger Website
Source control: GitHub
CI/CD: Jenkins
Container platform: Docker Swarm
Cloud provider: AWS
Container registry: Amazon ECR
Remote Jenkins access: Cloudflare Tunnel, subject to current configuration
