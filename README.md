# Jenkins CI/CD Docker Web Application

## Project Overview

This project contains a complete CI/CD pipeline using Jenkins.

The pipeline takes the application code from GitHub, builds it using Maven,
performs SonarQube code quality analysis, uploads the artifact to Nexus,
creates Docker images, scans the images using Trivy and finally deploys the
application using Docker Compose.

## CI/CD Flow

GitHub
   ↓
Jenkins
   ↓
Maven Build
   ↓
SonarQube
   ↓
Quality Gate
   ↓
Nexus
   ↓
Docker Images
   ↓
Trivy Scan
   ↓
Docker Compose
   ↓
Application + MySQL

---

## Technologies Used

- AWS EC2
- Jenkins
- Git / GitHub
- Maven
- SonarQube
- Nexus
- Docker
- Docker Compose
- Trivy
- Tomcat
- MySQL

---

## Repository Structure

```text
dockerwebapp/
│
├── Docker-app/
│   └── Dockerfile
│
├── Docker-db/
│   ├── Dockerfile
│   └── db_backup.sql
│
├── manifests/
│
├── src/
│
├── compose.yml
│
├── pom.xml
│
├── Jenkinsfile
│
└── README.md
````

---

# Prerequisites

Before running this project, install:

* Java 21
* Git
* Maven
* Docker
* Docker Compose
* Jenkins
* SonarQube
* Nexus
* Trivy

Make sure Docker is running:

```bash
docker --version
docker-compose --version
```

---

# 1. Clone the Repository

```bash
git clone https://github.com/SatishPolaka/dockerwebapp.git
```

Go inside the project:

```bash
cd dockerwebapp
```

---

# 2. Build the Application

The application is a Maven project.

Run:

```bash
mvn clean install
```

After successful build, check:

```bash
ls target/
```

The WAR file should be available:

```text
target/vprofile-v2.war
```

---

# 3. Build Docker Images

### Application Image

```bash
docker build -t mywebapp:v1 -f Docker-app/Dockerfile .
```

### Database Image

```bash
docker build -t mydb:v1 ./Docker-db
```

Check the images:

```bash
docker images
```

You should see:

```text
mywebapp    v1
mydb        v1
```

---

# 4. Docker Compose

The `compose.yml` file uses the same Docker images created above.

```yaml
services:
  myapplication:
    container_name: app-cont
    image: mywebapp:v1
    ports:
      - "1111:8080"
    depends_on:
      - devopsdb

  devopsdb:
    container_name: devopsdb
    image: mydb:v1
    ports:
      - "1112:3306"
    volumes:
      - userdata:/var/lib/mysql/
    environment:
      MYSQL_ROOT_PASSWORD: "devopspassword"

volumes:
  userdata:
```

Start the containers:

```bash
docker-compose up -d
```

Check:

```bash
docker-compose ps
```

or:

```bash
docker ps
```

---

# 5. Access the Application

The application container uses:

```text
8080
```

The host port is:

```text
1111
```

So access the application using:

```text
http://<EC2-PUBLIC-IP>:1111
```

The MySQL port is:

```text
1112
```

---

# 6. Jenkins CI/CD Pipeline

The Jenkins pipeline automates the complete process.

### Pipeline Stages

```text
CODE
 ↓
BUILD
 ↓
SQA
 ↓
QUALITY GATE
 ↓
ARTIFACT
 ↓
IMAGES
 ↓
TRIVY SCAN
 ↓
DEPLOY
```

### CODE

Jenkins checks out the GitHub repository.

### BUILD

Maven builds the application:

```bash
mvn clean install
```

### SQA

SonarQube performs code quality analysis.

### QUALITY GATE

Jenkins waits for the SonarQube Quality Gate result.

### ARTIFACT

The generated WAR file is uploaded to Nexus.

### IMAGES

Docker images are created:

```bash
docker build -t mywebapp:v1 -f Docker-app/Dockerfile .
docker build -t mydb:v1 ./Docker-db
```

### TRIVY SCAN

Docker images are scanned:

```bash
trivy image mywebapp:v1
trivy image mydb:v1
```

### DEPLOY

Docker Compose starts the application:

```bash
docker-compose up -d
```

---

# Important

The Docker image names used in `compose.yml` must match the images created
by the Jenkins pipeline.

The pipeline creates:

```text
mywebapp:v1
mydb:v1
```

So Compose also uses:

```yaml
image: mywebapp:v1
```

and:

```yaml
image: mydb:v1
```

If different names are used, Docker Compose may try to pull the image from
Docker Hub.

---

# Jenkins Requirements

In Jenkins, configure:

### Maven

Maven installation name:

```text
maven
```

### SonarQube

Configure the SonarQube server in Jenkins.

### Nexus

Add Nexus credentials in Jenkins Credentials.

### Docker

Jenkins user needs permission to access Docker:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins
```

Verify:

```bash
sudo -u jenkins docker ps
```

---

# Troubleshooting

## Docker Permission Denied

If Jenkins shows:

```text
permission denied while trying to connect to the Docker daemon socket
```

Run:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins
```

---

## Docker Build Target Directory Not Found

If you get:

```text
target: no such file or directory
```

Use:

```bash
docker build -t mywebapp:v1 -f Docker-app/Dockerfile .
```

The `.` is important because the `target` directory is outside `Docker-app`.

---

## Trivy Already Installed

The pipeline checks whether Trivy is already installed.

```bash
command -v trivy
```

If it exists, installation is skipped.

---

## Docker Compose Pull Access Denied

If you see:

```text
pull access denied for dbimage
```

check the image names:

```bash
docker images
```

Make sure `compose.yml` uses the same image names created by Jenkins.

---

# Final Result

After the pipeline completes successfully:

```text
GitHub
   ↓
Jenkins
   ↓
Maven
   ↓
SonarQube
   ↓
Quality Gate
   ↓
Nexus
   ↓
Docker
   ↓
Trivy
   ↓
Docker Compose
   ↓
Running Application
```

The complete application can then be accessed through:

```text
http://<EC2-PUBLIC-IP>:1111
```
