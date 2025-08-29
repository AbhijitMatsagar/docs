---
description: Deployment framework and approach
---

# Deployment Architecture

Continuous Integration/Continuous Deployment (CI/CD) software is the backbone of the modern DevOps environment. CI/CD bridges the gap between development and operations teams by automating the build, test, and deployment of applications.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

#### **CI/CD Workflow Architecture**

This CI/CD flow represents an automated deployment pipeline using AWS services including CodeCommit, CodePipeline, CodeBuild, CodeDeploy, and EC2.

**1. Source Stage (Code Commit)**

* Developers push code to **AWS CodeCommit**, which acts as the version control repository.
* A **CloudWatch Event** is triggered upon a new commit, which in turn initiates the CI/CD pipeline.

**2. CI Pipeline (AWS CodePipeline)**

* **AWS CodePipeline** orchestrates the entire CI/CD process.
* The pipeline includes three main stages:
  * **Build**
  * **Deploy to Staging**
  * **Deploy to Production**

**3. Build Stage**

* CodePipeline triggers **AWS CodeBuild** to compile the source code, run tests, and generate build artifacts.
* These build artifacts are then stored in an **S3 Bucket**.

**4. Deploy to Staging**

* Once the build is successful, the pipeline triggers **AWS CodeDeploy** to deploy the artifacts to **Staging EC2 instances** within a VPC.
* These EC2 instances are equipped with the **CodeDeploy Agent** to fetch the artifacts from S3 and perform the deployment.

**5. Deploy to Production**

* If the staging deployment is successful, another trigger initiates deployment to **Production EC2 instances** using **AWS CodeDeploy**.
* Like staging, production EC2 instances also use the **CodeDeploy Agent** to fetch the artifacts from the S3 bucket and perform the deployment.

**6. Environments**

* **Staging Environment**: Isolated environment for testing deployments before moving to production.
* **Production Environment**: Live environment that serves end users.

***

#### **Key AWS Services Used**

* **AWS CodeCommit**: Source control.
* **Amazon CloudWatch Events**: Pipeline trigger based on repository changes.
* **AWS CodePipeline**: Pipeline automation and orchestration.
* **AWS CodeBuild**: Builds and packages code into deployable artifacts.
* **AWS S3**: Stores build artifacts.
* **AWS CodeDeploy**: Handles deployment to EC2 instances.
* **Amazon EC2**: Hosts application in both staging and production environments.

***

This setup ensures that each code change is automatically built, tested, and deployed through a reliable, repeatable, and monitored process, increasing developer productivity and release confidence.

