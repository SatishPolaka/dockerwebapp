Absolutely. Below is the **complete README.md** in a more detailed and professional format, but I have kept the English **simple, natural and close to your way of explaining**.

You can directly use this as your GitHub `README.md`.

````markdown
# 🚀 End-to-End Jenkins CI/CD Pipeline with Docker

## 📌 Project Overview

This project is an end-to-end CI/CD pipeline that I implemented using Jenkins.

The main purpose of this project is to automate the complete application deployment process starting from source code checkout from GitHub and ending with application deployment using Docker Compose.

The pipeline covers the complete flow:

- Source code checkout from GitHub
- Maven application build
- SonarQube code quality analysis
- SonarQube Quality Gate
- Artifact upload to Nexus
- Docker image creation
- Trivy security scanning
- Docker Compose deployment
- Application and MySQL database deployment

The complete flow is automated through Jenkins.

---

# 🏗️ CI/CD Pipeline Flow

```text
                         GitHub
                            |
                            ↓
                         Jenkins
                            |
                            ↓
                     CODE CHECKOUT
                            |
                            ↓
                       MAVEN BUILD
                            |
                            ↓
                        SONARQUBE
                            |
                            ↓
                      QUALITY GATE
                            |
                            ↓
                          NEXUS
                            |
                            ↓
                    DOCKER IMAGES
                            |
                            ↓
                       TRIVY SCAN
                            |
                            ↓
                    DOCKER COMPOSE
                            |
                            ↓
                  APPLICATION + DATABASE
````

---

# 🛠️ Technologies Used

| Category             | Technology     |
| -------------------- | -------------- |
| Cloud                | AWS EC2        |
| Source Code          | Git / GitHub   |
| CI/CD                | Jenkins        |
| Build Tool           | Maven          |
| Code Quality         | SonarQube      |
| Artifact Repository  | Nexus          |
| Containerization     | Docker         |
| Container Deployment | Docker Compose |
| Security Scanning    | Trivy          |
| Application Server   | Apache Tomcat  |
| Database             | MySQL          |
| Operating System     | Linux          |

---

# 📂 Project Structure

The repository contains the application code, Docker files, Docker Compose configuration and Jenkins pipeline.

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
```

### Important files

### `pom.xml`

This is the Maven project file.

It contains the project details and dependencies required to build the application.

### `Docker-app/Dockerfile`

This Dockerfile is used to create the application Docker image.

### `Docker-db/Dockerfile`

This Dockerfile is used to create the MySQL database image.

### `db_backup.sql`

This contains the database backup/data required for the database container.

### `compose.yml`

This file is used to run the application and database containers together.

### `Jenkinsfile`

This contains the Jenkins pipeline stages used to automate the CI/CD process.

---

# 🔧 Prerequisites

Before running this project, make sure the following tools are installed and configured.

```text
Java
Git
Maven
Docker
Docker Compose
Jenkins
SonarQube
Nexus
Trivy
```

For the Jenkins server, Docker access is also required because Jenkins will build Docker images and run Docker Compose commands.

---

# 1. Clone the Repository

First clone the project from GitHub.

```bash
git clone https://github.com/SatishPolaka/dockerwebapp.git
```

Go inside the project directory:

```bash
cd dockerwebapp
```

Check the project files:

```bash
ls
```

You should see files and directories like:

```text
Docker-app
Docker-db
manifests
src
compose.yml
pom.xml
Jenkinsfile
README.md
```

---

# 2. Build the Application Using Maven

The application is a Maven project.

Before creating the application Docker image, we need to build the application using Maven.

Run:

```bash
mvn clean install
```

### What happens here?

Maven downloads the required dependencies, compiles the application and creates the WAR file.

After a successful build, check the `target` directory:

```bash
ls target/
```

The application WAR file will be generated:

```text
target/vprofile-v2.war
```

This WAR file will later be copied into the Tomcat Docker image.

---

# 3. Application Dockerfile

The application Dockerfile is present inside:

```text
Docker-app/Dockerfile
```

The Dockerfile:

```dockerfile
FROM tomcat:8-jre11

RUN rm -rf /usr/local/tomcat/webapps/*

COPY target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

CMD ["catalina.sh", "run"]
```

### Explanation

### `FROM`

```dockerfile
FROM tomcat:8-jre11
```

We are using Tomcat as the base image because the application is packaged as a WAR file.

### Remove default applications

```dockerfile
RUN rm -rf /usr/local/tomcat/webapps/*
```

The default Tomcat applications are removed so that our application can be deployed directly.

### Copy WAR file

```dockerfile
COPY target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
```

The WAR file generated by Maven is copied into the Tomcat `webapps` directory.

It is renamed to `ROOT.war`, so the application can be accessed from the root path.

### Start Tomcat

```dockerfile
CMD ["catalina.sh", "run"]
```

This starts Tomcat when the container starts.

---

# 4. Build the Application Docker Image

Now we can create the application Docker image.

Run:

```bash
docker build -t mywebapp:v1 -f Docker-app/Dockerfile .
```

### Why are we using `-f`?

Our Dockerfile is inside:

```text
Docker-app/Dockerfile
```

So we specify the Dockerfile using:

```bash
-f Docker-app/Dockerfile
```

### Why are we using `.` at the end?

The `.` means the current project directory is used as the Docker build context.

This is important because the Maven `target` directory is present in the project root.

The Dockerfile needs access to:

```text
target/vprofile-v2.war
```

Check the image:

```bash
docker images
```

You should see:

```text
mywebapp    v1
```

---

# 5. Build the Database Docker Image

The database Dockerfile is present inside:

```text
Docker-db/
```

Build the database image:

```bash
docker build -t mydb:v1 ./Docker-db
```

Here:

```text
mydb
```

is the image name.

```text
v1
```

is the image tag.

Check the images:

```bash
docker images
```

You should see both images:

```text
mywebapp    v1
mydb        v1
```

---

# 6. Docker Compose

After creating the application and database images, we need to run both containers.

Instead of manually running each container separately, Docker Compose is used to define and run them together.

The project contains:

```text
compose.yml
```

The Compose file contains the application and database services.

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

---

# 7. Application Container

The application service uses:

```yaml
image: mywebapp:v1
```

This is the Docker image we created earlier.

The port mapping is:

```yaml
ports:
  - "1111:8080"
```

This means:

```text
EC2 Host Port     Container Port
     1111    →        8080
```

Tomcat is running on port `8080` inside the container.

We are exposing it through port `1111` on the EC2 server.

---

# 8. Database Container

The database service uses:

```yaml
image: mydb:v1
```

The port mapping is:

```yaml
ports:
  - "1112:3306"
```

This means:

```text
EC2 Host Port     Container Port
     1112    →        3306
```

MySQL normally runs on port `3306`.

---

# 9. Docker Volume

The database service uses a Docker volume:

```yaml
volumes:
  - userdata:/var/lib/mysql/
```

The volume is mounted to:

```text
/var/lib/mysql/
```

This is the directory where MySQL stores its database data.

Using a Docker volume means the database data is stored separately from the container.

The volume is defined at the bottom:

```yaml
volumes:
  userdata:
```

Docker will manage this volume for us.

---

# 10. Start the Application

After building the images, start the application and database using:

```bash
docker-compose up -d
```

The `-d` option runs the containers in the background.

Check the containers:

```bash
docker-compose ps
```

You can also check:

```bash
docker ps
```

You should see:

```text
app-cont
devopsdb
```

both running.

---

# 11. Access the Application

The application is running inside the Tomcat container on port:

```text
8080
```

We mapped it to EC2 port:

```text
1111
```

So the application can be accessed using:

```text
http://<EC2-PUBLIC-IP>:1111
```

For example:

```text
http://<YOUR-EC2-PUBLIC-IP>:1111
```

Make sure the AWS Security Group allows inbound traffic on port `1111`.

---

# 12. Jenkins CI/CD Pipeline

The manual process works, but the main purpose of this project is to automate this process using Jenkins.

The Jenkins pipeline performs the complete flow automatically.

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

---

# 13. CODE Stage

The first stage checks out the source code from GitHub.

Example:

```groovy
stage('CODE') {
    steps {
        git 'https://github.com/SatishPolaka/dockerwebapp.git'
    }
}
```

Jenkins downloads the project into its workspace.

After this stage, Jenkins has all the application source code, Docker files, Compose file and Maven configuration required for the next stages.

---

# 14. BUILD Stage

The next stage builds the application using Maven.

```groovy
stage('BUILD') {
    steps {
        sh 'mvn clean install'
    }
}
```

This generates:

```text
target/vprofile-v2.war
```

This WAR file is required to create the application Docker image.

---

# 15. SQA Stage

After building the application, SonarQube is used for code quality analysis.

The purpose of this stage is to analyse the source code and identify code quality issues.

The Jenkins pipeline sends the Maven project for analysis to SonarQube.

Example:

```groovy
stage('SQA') {
    steps {
        withSonarQubeEnv('mysonar') {
            sh '''
                mvn clean verify \
                    org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                    -Dsonar.projectKey=my-project_test \
                    -Dsonar.projectName='my-project_test' \
                    -Dsonar.host.url=http://<SONARQUBE-IP>:9000 \
                    -Dsonar.token=$SONAR_AUTH_TOKEN
            '''
        }
    }
}
```

The SonarQube token should be stored securely in Jenkins Credentials.

It should not be directly written inside the Jenkinsfile.

---

# 16. Quality Gate

After SonarQube analysis, Jenkins waits for the SonarQube Quality Gate result.

Example:

```groovy
stage('Qualitygates') {
    steps {
        waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
    }
}
```

The Quality Gate helps us check whether the project meets the configured code quality conditions.

---

# 17. Nexus Artifact Repository

After the application build, the generated WAR file can be uploaded to Nexus.

The artifact details used in this project are:

```text
Group ID     : com.visualpathit
Artifact ID  : vprofile
Version      : v2
Artifact     : vprofile-v2.war
```

Nexus is used as an artifact repository so that the generated application package can be stored and managed separately from the source code.

---

# 18. Docker Image Creation in Jenkins

After the artifact stage, Jenkins creates the Docker images.

Application image:

```bash
docker build -t mywebapp:v1 -f Docker-app/Dockerfile .
```

Database image:

```bash
docker build -t mydb:v1 ./Docker-db
```

After this stage, Jenkins has:

```text
mywebapp:v1
mydb:v1
```

These images are then used by Docker Compose for deployment.

---

# 19. Trivy Image Scan

Before deployment, the Docker images are scanned using Trivy.

Trivy is used to identify vulnerabilities inside the Docker images.

The pipeline runs:

```bash
trivy image mywebapp:v1
trivy image mydb:v1
```

This gives us information about vulnerabilities found in the images.

---

# 20. Docker Permission for Jenkins

Jenkins runs using the Linux user:

```text
jenkins
```

The Jenkins user needs permission to access Docker because the pipeline runs commands such as:

```bash
docker build
docker images
docker-compose up -d
```

If Jenkins does not have Docker permission, the pipeline can fail with an error similar to:

```text
permission denied while trying to connect to the Docker daemon socket
```

To give Jenkins Docker access, add the Jenkins user to the Docker group:

```bash
sudo usermod -aG docker jenkins
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

Verify:

```bash
sudo -u jenkins docker ps
```

If the command works without a permission error, Jenkins can access Docker.

The `sudo` commands are used during server configuration. We are not giving Jenkins general `sudo` access here. We are adding Jenkins to the Docker group so that the Jenkins process can access Docker.

---

# 21. Docker Compose Deployment from Jenkins

After the Docker images are created and scanned, Jenkins deploys the application using Docker Compose.

```bash
docker-compose up -d
```

This starts:

```text
Application Container
        +
Database Container
```

Check the deployment:

```bash
docker-compose ps
```

and:

```bash
docker ps
```

---

# 22. Important Docker Image Names

One important point in this project is that the image names in `compose.yml` must match the image names created by Jenkins.

Jenkins creates:

```text
mywebapp:v1
mydb:v1
```

So `compose.yml` also uses:

```yaml
image: mywebapp:v1
```

and:

```yaml
image: mydb:v1
```

If the names are different, Docker Compose may try to pull the image from Docker Hub.

For example, if Compose contains:

```yaml
image: dbimage
```

but the local image is:

```text
mydb:v1
```

Docker will not find the expected local image and may try to pull `dbimage:latest`.

---

# 23. Jenkins Maven Configuration

The Jenkins pipeline uses Maven.

The Jenkinsfile contains:

```groovy
tools {
    maven 'maven'
}
```

Because of this, Jenkins needs a Maven installation configured with the name:

```text
maven
```

This can be configured from:

```text
Manage Jenkins
    ↓
Tools
    ↓
Maven installations
```

The name should match the name used in the Jenkinsfile.

---

# 24. SonarQube Configuration in Jenkins

Jenkins needs to know where SonarQube is running.

In Jenkins:

```text
Manage Jenkins
    ↓
System
    ↓
SonarQube Servers
```

Configure the SonarQube server name used by the pipeline:

```text
mysonar
```

The URL should point to the SonarQube server.

The SonarQube authentication token should be stored in Jenkins Credentials instead of keeping the token directly inside the Jenkinsfile.

---

# 25. Nexus Configuration

Nexus is used to store the generated WAR artifact.

The Jenkins pipeline uploads:

```text
vprofile-v2.war
```

with:

```text
Group ID    : com.visualpathit
Artifact ID : vprofile
Version     : v2
```

The Nexus repository used in this project is:

```text
dockerapp
```

The Nexus username and password should be stored in Jenkins Credentials.

---

# 26. Trivy Installation

Trivy needs to be available on the Jenkins machine before the scan stage runs.

Check whether Trivy is already installed:

```bash
trivy --version
```

or:

```bash
command -v trivy
```

If Trivy is already installed, there is no need to install it again.

The pipeline can check this before installation.

Example:

```bash
if command -v trivy >/dev/null 2>&1; then
    echo "Trivy is already installed"
    trivy --version
else
    echo "Trivy not found. Installing..."
fi
```

This avoids trying to install the same package again on every pipeline run.

---

# 27. Common Issues Faced

During the project implementation, some issues were faced and fixed.

## Docker Permission Issue

### Error

```text
permission denied while trying to connect to the Docker daemon socket
```

### Reason

The Jenkins user did not have permission to access Docker.

### Solution

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins
```

---

## Docker Build Context Issue

Initially, building the application image from the `Docker-app` directory caused an issue because the WAR file was available under the project root `target` directory.

Instead of changing the Dockerfile, the build context was changed.

Use:

```bash
docker build -t mywebapp:v1 -f Docker-app/Dockerfile .
```

The `.` makes the complete project directory available as the Docker build context.

---

## Docker Compose Image Name Issue

If Compose uses an image name that does not exist locally, Docker may try to pull the image from Docker Hub.

For example:

```yaml
image: dbimage
```

while the actual image is:

```text
mydb:v1
```

The solution is to use the same image name and tag:

```yaml
image: mydb:v1
```

---

## Trivy Version Issue

When installing Trivy, the download URL and package version must match.

If the URL points to one version but the package filename contains another version, the download can fail.

So the Trivy version and download file name should always match.

---

# 28. Complete Manual Flow

If you want to run the project manually without Jenkins, the basic flow is:

### Clone

```bash
git clone https://github.com/SatishPolaka/dockerwebapp.git
cd dockerwebapp
```

### Maven Build

```bash
mvn clean install
```

### Build Application Image

```bash
docker build -t mywebapp:v1 -f Docker-app/Dockerfile .
```

### Build Database Image

```bash
docker build -t mydb:v1 ./Docker-db
```

### Check Images

```bash
docker images
```

### Start Application

```bash
docker-compose up -d
```

### Check Containers

```bash
docker-compose ps
docker ps
```

### Scan Images

```bash
trivy image mywebapp:v1
trivy image mydb:v1
```

---

# 29. Complete Automated Flow

With Jenkins, the complete process is automated:

```text
Developer pushes code
        ↓
      GitHub
        ↓
      Jenkins
        ↓
   Code Checkout
        ↓
    Maven Build
        ↓
    SonarQube
        ↓
   Quality Gate
        ↓
      Nexus
        ↓
  Docker Image Build
        ↓
    Trivy Scan
        ↓
 Docker Compose Deploy
        ↓
 Running Application
```

This removes the need to manually perform each step after the pipeline is configured.

---

# 30. Final Result

After successful pipeline execution:

* Application code is checked out from GitHub.
* Maven builds the application.
* SonarQube checks the code quality.
* Quality Gate result is checked.
* WAR artifact is uploaded to Nexus.
* Application and database Docker images are created.
* Docker images are scanned using Trivy.
* Docker Compose deploys the application and database.
* Application becomes available through the configured EC2 port.

Application URL:

```text
http://<EC2-PUBLIC-IP>:1111
```
