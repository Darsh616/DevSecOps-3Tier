# DevSecOps 3-Tier Project

## Overview
This project implements a **DevSecOps pipeline** for a **3-tier application**, integrating security tools like **SonarQube, OWASP Dependency Check, and Trivy** to ensure code quality and vulnerability scanning. The pipeline is managed using **Jenkins, Docker, and Docker Compose**.

## Prerequisites
Before setting up the project, ensure you have the following installed on your system:
- **Docker & Docker Compose**
- **Maven**
- **OpenJDK 17**
- **Jenkins** with the required plugins
- **SonarQube Server**
- **OWASP Dependency Check**
- **Trivy Security Scanner**

## SonarQube using Docker
- `docker run -itd --name sonarqube-server -p 9001:9000 sonarqube:lts-community `

## Jenkins Setup
### 1. Install Required Plugins
Go to **Manage Jenkins → Plugin Manager** and install:
- **SonarQube Scanner**
- **Sonar Quality Gateway**
- **Docker**
- **OWASP Dependency Check**

### 2. Configure Global Credentials
1. **Create a Personal Access Token (PAT) for SonarQube**:
   - Navigate to **SonarQube → Administrator → Users**
   - Generate a token and store it in **Jenkins credentials**

2. **Create a Webhook in SonarQube**:
   - Go to **SonarQube → Webhooks**
   - Name: `Jenkins`
   - URL: `http://localhost:8080/sonarqube-webhook/`
   - Click **Create**

3. **Configure SonarQube in Jenkins**:
   - Navigate to **Manage Jenkins → Configure System**
   - Add a **SonarQube Installation**:
     - Name: `Sonar`
     - URL: `http://localhost:9000`
     - Select the SonarQube token

### 3. Install Required Tools in Jenkins
1. **SonarQube Scanner**:
   - Navigate to **Global Tool Configuration**
   - Add **SonarQube Scanner** installation
   - Name: `Sonar`
   - Select **Install automatically**

2. **OWASP Dependency Check**:
   - Navigate to **Global Tool Configuration**
   - Add **OWASP Dependency Check**
   - Name: `OWASP`
   - Select **Install automatically**

## Running SonarQube and Trivy
Run the following commands to set up SonarQube and Trivy:
```bash
# Start SonarQube Server using Docker
sudo docker run -itd --name sonarqube-server -p 9001:9000 sonarqube:lts-community

# Install Trivy Security Scanner
wget https://github.com/aquasecurity/trivy/releases/download/v0.34.0/trivy_0.34.0_Linux-64bit.tar.gz
tar -xvf trivy_0.34.0_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/
trivy --version
```

## Jenkins Pipeline Configuration
Create a **Jenkinsfile** in your Git repository with the following pipeline:

```groovy
pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
        TRIVY_HOME = '/usr/local/bin'
        PROJECT_PATH = '/var/lib/jenkins/workspace/DevSec'
    }

    stages {
        stage('1. Cloning the Code') {
            steps {
                git url: "https://github.com/krishnaacharyaa/wanderlust.git", branch: "devops"
            }
        }

        stage('2. SonarQube Quality Analysis') {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Wanderlust -Dsonar.projectKey=Wanderlust"
                }
            }
        }

        stage('3. OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: "-- scan ./", odcInstallation: "OWASP"
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("4. Wait for SonarQube") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('5. Trivy Vulnerability Scan') {
            steps {
                sh "$TRIVY_HOME/trivy fs ${PROJECT_PATH} --format table -o trivy-fs-report.html"
            }
        }

        stage('6. Deploy using Docker') {
            steps {
                sh "docker-compose up"
            }
        }
    }
}
```

## Running the Pipeline
1. **Push the `Jenkinsfile`** to your repository.
2. **Create a Jenkins job**:
   - Select **Pipeline** type.
   - Configure the pipeline to use **SCM (Git)** and enter your repository URL.
   - Save and **Run the pipeline**.

## Outputs and Reports
- **SonarQube Dashboard**: Displays code quality results.
- **OWASP Dependency Check Report**: Identifies vulnerable dependencies.
- **Trivy Report**: Detects vulnerabilities in your files and dependencies.
- **Docker Deployment**: Launches the 3-tier application.

![image](https://github.com/user-attachments/assets/6a92c224-ae47-4fa5-8076-702773dc2bfc)

![image](https://github.com/user-attachments/assets/236bd54d-2f28-4278-8d8a-dc8bb57db44e)

![image](https://github.com/user-attachments/assets/eaa4deaf-597a-457d-840b-34e4d27b56d7)

