pipeline {
    agent {
        node {
            label 'prod'
        }
    }

    tools {
        maven 'mymaven'
    }

    stages {
        stage('CODE') {
            steps {
                git branch: 'app', url: 'https://github.com/SatishPolaka/dockerwebapp.git'
            }
        }

        stage('BUILD') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SONARQUBE') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh ''' mvn clean verify \
                        org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=app_java'''
                }
            }
        }
        
        stage ('QUALITYGATES') {
            steps {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar-password'
            }
        }
        
        stage ('ARTIFACT UPLOADER') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'Nexsus-uploader', groupId: 'com.visualpathit', nexusUrl: '3.236.200.138:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'repo_appjava', version: 'v2'
            }
        }
        
        stage ('IMAGES') {
            steps {
                sh 'cp -r target Docker-app'
                sh 'docker build -t appimage Docker-app'
                sh 'docker build -t dbimage Docker-db'
            }
        }
        
        stage ('SCAN IMAGES') {
            steps {
                sh 'trivy image appimage >> app-image-trivyreport.txt'
                sh 'trivy image dbimage >> db-image-trivyreport.txt'
            }
        }
        
        stage ('REGISTRY') {
            steps {
                withDockerRegistry(credentialsId: 'DockerHub') {
                    sh 'docker tag appimage satishpolaka/javaapp:v1'
                    sh 'docker tag dbimage satishpolaka/db:v1'
                    sh 'docker push satishpolaka/javaapp:v1'
                    sh 'docker push satishpolaka/db:v1'
                }
            }
        }
        
        stage ('DEPLOY') {
            steps {
                sh 'docker stack deploy myapp --compose-file=compose.yml'
            }
        }
    }
}
