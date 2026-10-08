# End-to-End Java CI/CD Pipeline with Jenkins, Docker & Docker Swarm

## Project Overview

This project demonstrates an end-to-end CI/CD pipeline for a Java web application using Jenkins.

The pipeline automates the complete process from pulling the source code from GitHub to deploying the application using Docker Swarm.

The pipeline includes:

* GitHub
* Jenkins
* Maven
* SonarQube
* SonarQube Quality Gate
* Nexus Repository
* Docker
* Trivy
* Docker Hub
* Docker Swarm

## Architecture

```text
                         GitHub
                            |
                            v
                         Jenkins
                            |
                            v
                      Maven Build
                            |
                            v
                       SonarQube
                            |
                            v
                      Quality Gate
                            |
                            v
                         Nexus
                            |
                            v
                    Docker Image Build
                            |
                            v
                       Trivy Scan
                            |
                            v
                       Docker Hub
                            |
                            v
                  Docker Swarm Manager
                            |
                            v
                       Worker Node
                            |
                            v
                      Application
```

## AWS Infrastructure

The environment is configured using separate EC2 instances:

```text
Jenkins + SonarQube
        |
        +-- Jenkins Server
        +-- SonarQube running as Docker container

Nexus
        |
        +-- Nexus Repository

Worker
        |
        +-- Jenkins Worker
        +-- Docker workload

Manager
        |
        +-- Docker Swarm Manager
```

## Technologies Used

| Technology   | Purpose                             |
| ------------ | ----------------------------------- |
| GitHub       | Source code management              |
| Jenkins      | CI/CD automation                    |
| Maven        | Java application build              |
| SonarQube    | Code quality analysis               |
| Nexus        | Artifact repository                 |
| Docker       | Containerisation                    |
| Trivy        | Docker image vulnerability scanning |
| Docker Hub   | Docker image registry               |
| Docker Swarm | Container orchestration             |
| AWS EC2      | Infrastructure                      |

---

# Jenkins Pipeline

The Jenkins pipeline contains the following stages:

```text
CODE
  ↓
BUILD
  ↓
SONARQUBE
  ↓
QUALITYGATES
  ↓
ARTIFACT UPLOADER
  ↓
IMAGES
  ↓
SCAN IMAGES
  ↓
REGISTRY
  ↓
DEPLOY
```

The pipeline runs on the Jenkins worker configured with the label:

```groovy
label 'prod'
```

---

# 1. CODE

The pipeline starts by checking out the application source code from GitHub.

```groovy
git branch: 'app',
    url: 'https://github.com/SatishPolaka/dockerwebapp.git'
```

The `app` branch is checked out into the Jenkins workspace.

At this stage, Jenkins gets the latest application source code that will be used for the remaining pipeline stages.

---

# 2. BUILD

Maven is configured as a Jenkins tool:

```groovy
tools {
    maven 'mymaven'
}
```

The application is built using:

```bash
mvn clean install
```

This cleans the previous build files, compiles the application, runs the required Maven lifecycle steps and generates the application artifact.

The WAR file is generated inside the `target` directory.

Example:

```text
target/vprofile-v2.war
```

---

# 3. SONARQUBE

After the Maven build, the application is analysed using SonarQube.

SonarQube is running inside a Docker container on the Jenkins/SonarQube EC2 instance.

The Jenkins pipeline uses the configured SonarQube server:

```groovy
withSonarQubeEnv('mysonar')
```

The SonarQube analysis is executed using:

```bash
mvn clean verify \
org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
-Dsonar.projectKey=app_java
```

The full SonarQube Maven plugin coordinates are used here so Maven can directly resolve the SonarQube plugin.

The analysis results are sent to the SonarQube server.

---

# 4. QUALITYGATES

After the SonarQube analysis, Jenkins waits for the SonarQube Quality Gate result.

```groovy
waitForQualityGate abortPipeline: true,
    credentialsId: 'sonar-password'
```

The Quality Gate controls whether the pipeline should continue.

```text
SonarQube Analysis
        |
        v
   Quality Gate
        |
   +----+----+
   |         |
   OK       Failed
   |         |
   v         v
Continue    Stop
Pipeline   Pipeline
```

If the Quality Gate is successful, the pipeline continues to the Nexus artifact upload stage.

If the Quality Gate fails, the pipeline is stopped.

---

# 5. ARTIFACT UPLOADER

After the Quality Gate passes, the generated WAR file is uploaded to Nexus Repository.

The pipeline uploads:

```text
target/vprofile-v2.war
```

The Nexus configuration used by the pipeline contains:

```text
Nexus Version: Nexus 3
Protocol: HTTP
Repository: repo_appjava
Group ID: com.visualpathit
Artifact ID: vprofile
Version: v2
```

The artifact is stored in Nexus so that the build output is available independently from the source code.

The flow is:

```text
Maven Build
     |
     v
WAR File
     |
     v
SonarQube Quality Gate
     |
     v
Nexus Repository
```

---

# 6. IMAGES

After the artifact is uploaded successfully, Docker images are created.

First, the application build output is copied into the Docker application directory:

```bash
cp -r target Docker-app
```

Then the application Docker image is created:

```bash
docker build -t appimage Docker-app
```

The database Docker image is also created:

```bash
docker build -t dbimage Docker-db
```

At the end of this stage, the Jenkins worker has two Docker images:

```text
appimage
dbimage
```

---

# 7. SCAN IMAGES

The Docker images are scanned using Trivy.

Application image:

```bash
trivy image appimage >> app-image-trivyreport.txt
```

Database image:

```bash
trivy image dbimage >> db-image-trivyreport.txt
```

The scan results are stored in:

```text
app-image-trivyreport.txt
db-image-trivyreport.txt
```

This stage is used to identify known vulnerabilities in the Docker images before pushing them to the registry.

---

# 8. REGISTRY

After the Docker images are scanned, they are tagged with the Docker Hub repository names.

Application image:

```bash
docker tag appimage satishpolaka/javaapp:v1
```

Database image:

```bash
docker tag dbimage satishpolaka/db:v1
```

The images are then pushed to Docker Hub:

```bash
docker push satishpolaka/javaapp:v1
```

```bash
docker push satishpolaka/db:v1
```

The Jenkins pipeline uses Docker Hub credentials configured in Jenkins:

```groovy
withDockerRegistry(credentialsId: 'DockerHub')
```

The flow is:

```text
Local Docker Image
       |
       v
Docker Tag
       |
       v
Docker Hub
       |
       +-- javaapp:v1
       |
       +-- db:v1
```

---

# 9. DEPLOY

The final stage deploys the application using Docker Swarm.

The application stack is deployed using:

```bash
docker stack deploy -c compose.yml myapp
```

Here:

* `compose.yml` contains the service configuration.
* `myapp` is the Docker Swarm stack name.

The Docker Swarm manager receives the deployment request and schedules the services on the available Swarm nodes.

The deployment flow is:

```text
Docker Hub
     |
     v
Swarm Manager
     |
     v
Worker Node
     |
     v
Application Containers
```

---

# Docker Swarm Verification

After deployment, the Swarm can be checked from the manager node.

Check the Swarm nodes:

```bash
docker node ls
```

Check the deployed stacks:

```bash
docker stack ls
```

Check the services:

```bash
docker stack services myapp
```

Check the running tasks:

```bash
docker stack ps myapp
```

On the worker node, running containers can be checked using:

```bash
docker ps
```

The application can then be accessed through the configured application endpoint.

---

# Complete CI/CD Flow

The complete pipeline works as follows:

```text
Developer
    |
    v
GitHub
    |
    v
Jenkins
    |
    v
CODE
    |
    v
Maven BUILD
    |
    v
SonarQube Analysis
    |
    v
Quality Gate
    |
    +------ Failed ------> Pipeline Stops
    |
    | Passed
    v
Nexus Artifact Upload
    |
    v
Docker Image Build
    |
    v
Trivy Image Scan
    |
    v
Docker Hub
    |
    v
Docker Swarm Manager
    |
    v
Worker Node
    |
    v
Application
```

# Jenkins Pipeline Stages Summary

| Stage             | Activity                                     |
| ----------------- | -------------------------------------------- |
| CODE              | Clone application code from GitHub           |
| BUILD             | Build Java application using Maven           |
| SONARQUBE         | Analyse application code using SonarQube     |
| QUALITYGATES      | Validate SonarQube Quality Gate              |
| ARTIFACT UPLOADER | Upload WAR file to Nexus                     |
| IMAGES            | Build application and database Docker images |
| SCAN IMAGES       | Scan Docker images using Trivy               |
| REGISTRY          | Push Docker images to Docker Hub             |
| DEPLOY            | Deploy application using Docker Swarm        |

# Project Result

This project provides an automated CI/CD workflow where a code change can go through the complete process:

```text
Source Code
     ↓
Build
     ↓
Code Quality
     ↓
Quality Gate
     ↓
Artifact Management
     ↓
Container Build
     ↓
Security Scan
     ↓
Container Registry
     ↓
Docker Swarm
     ↓
Application Deployment
```
