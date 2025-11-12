# AWS CI/CD Pipeline for Dockerized 2048 Game

> **Disclaimer:**  
> This project was created as part of a cloud development learning environment to demonstrate CI/CD automation using AWS services.  
> The open-source code for the 2048 game is used for educational purposes only.

---

## Overview

This project focuses on building a **CI/CD pipeline** for deploying a Dockerized version of the open-source **2048 game** using AWS services.  
The pipeline automates the process of building, testing, and deploying the game container to **Amazon ECS**, ensuring scalable and seamless deployments.

Through this project, you’ll gain practical experience integrating **AWS CodePipeline**, **CodeBuild**, **ECR**, and **ECS**—core components of modern DevOps workflows in the AWS ecosystem.

### Key Objectives
- **Automate** the build and deployment of a containerized web application.  
- **Deploy** the 2048 game to **Amazon ECS (Fargate)** using a CI/CD pipeline.  
- **Integrate** multiple AWS services for continuous integration and continuous delivery.  

---

## Architecture Diagram

![AWS CI/CD Pipeline for 2048 Game](https://github.com/user-attachments/assets/0bff535a-a47f-4d31-9852-b622aada9971)

*Figure 1: End-to-end CI/CD architecture for deploying the Dockerized 2048 game using AWS CodePipeline, CodeBuild, ECR, and ECS.*

### Workflow Summary
1. **Source Stage:**  
   The 2048 game source code is hosted in a **GitHub repository**. CodePipeline automatically detects commits and triggers the build process.
2. **Build Stage:**  
   **AWS CodeBuild** builds a Docker image for the game, tags it, and prepares it for deployment.
3. **Push to ECR:**  
   The Docker image is pushed to **Amazon Elastic Container Registry (ECR)**.
4. **Deploy Stage:**  
   **AWS CodeDeploy** (via CodePipeline) updates the **Amazon ECS** service to pull and run the new container image.

---

## Services Used

### 1. AWS CodePipeline
- **Purpose:** Automates the CI/CD process by orchestrating code changes from source to deployment.
- **Functionality:** Detects code changes from GitHub, triggers the build process, and automates ECS deployment.

### 2. AWS CodeBuild
- **Purpose:** Handles the **build phase** of the pipeline.
- **Functionality:** Builds and tags the Docker image, then pushes it to Amazon ECR for deployment.

### 3. Amazon Elastic Container Registry (ECR)
- **Purpose:** Acts as a secure container registry to store and manage Docker images.
- **Functionality:** Stores versioned container images that are deployed to ECS.

### 4. Amazon Elastic Container Service (ECS)
- **Purpose:** Runs the Dockerized 2048 game using the **Fargate launch type**.
- **Functionality:** Manages containerized workloads and automatically scales as needed.

### 5. AWS Identity and Access Management (IAM)
- **Purpose:** Provides secure access control across AWS services.
- **Functionality:** Assigns roles and permissions for CodePipeline, CodeBuild, and ECS to interact securely.

---

## Implementation Steps

### 1. Clone the 2048 Game Repository
Clone the open-source project to your local environment:
```bash
git clone https://github.com/techwithlucy/ztc-projects.git
cd ztc-projects/projects/advanced-aws-projects/project-1
```
This repository contains the 2048 game source code and a preconfigured Dockerfile.

### 2. Build and Push the Docker Image to Amazon ECR

Check the Dockerfile
```dockerfile
# Use the official Nginx image as the base
FROM nginx:latest

# Copy the 2048 game files to the Nginx web root
COPY . /usr/share/nginx/html

# Expose the default Nginx HTTP port
EXPOSE 80

# Start Nginx when the container starts
CMD ["nginx", "-g", "daemon off;"]
```
Build and Tag the Docker Image
```
docker build -t 2048-game .
```
Tag for ECR

Create an ECR repository (e.g., 2048-game-repo) in the AWS Console, then tag your image:
```
docker tag 2048-game:latest <ECR_URI>:latest
```
Authenticate and Push to ECR
```
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <ECR_URI>
docker push <ECR_URI>:latest
```
Verify the image in the ECR Console to ensure it was uploaded successfully.

### 3. Set Up the ECS Cluster

- Create an ECS Cluster using the Fargate launch type.

- Define a Task Definition referencing the image in ECR.

- Create a Service within ECS to manage and run the 2048 game container.

### 4. Configure AWS CodeBuild
- Create a buildspec.yml file to automate Docker image building and pushing.

Example `buildspec.yml:`
```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <ECR_URI>
  build:
    commands:
      - echo Building the Docker image...
      - docker build -t 2048-game .
      - docker tag 2048-game:latest <ECR_URI>:latest
  post_build:
    commands:
      - echo Pushing the Docker image...
      - docker push <ECR_URI>:latest
      - echo Writing imagedefinitions.json...
      - printf '[{"name":"2048-game","imageUri":"%s"}]' "<ECR_URI>:latest" > imagedefinitions.json
artifacts:
  files:
    - imagedefinitions.json
```

### 5. Set Up CodePipeline
- Configure AWS CodePipeline with three stages:
  1. Source: GitHub repository (triggers on new commits).
  2. Build: AWS CodeBuild project to create and push Docker images.
  3. Deploy: Amazon ECS service deployment using the new image from ECR.

 ### 6. Access the Application
 Once the deployment completes, access the game via the ECS task public IP address over port 80. You should see the 2048 game running live, served through Nginx inside your Docker container.

 Example:
 ```
http://<Public-IP>:80
```

---
## Conclusion

This project demonstrates a fully automated CI/CD pipeline on AWS for containerized applications.
By integrating CodePipeline, CodeBuild, ECR, and ECS, the pipeline provides a continuous and reliable deployment process for the 2048 game.

This setup reflects real-world DevOps practices, emphasizing automation, scalability, and security — key principles in modern cloud-based software delivery.

![image](https://github.com/user-attachments/assets/d2bf7f4e-3253-4fb7-b860-9e4a87a9cee5)
![image](https://github.com/user-attachments/assets/d9b84e41-d600-4288-97e7-cde207bce074)
