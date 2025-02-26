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
