pipeline {
    agent none

    environment {
        REPO_URL = 'https://github.com/mhykari/jenkins-test.git'
        PROJECT_DIR = 'java-api'
        IMAGE_NAME = 'jaaava-api'
        CONTAINER_NAME = 'java-api-container'
    }

    stages {
        stage('Checkout Code') {
            agent any
            steps {
                echo "Cloning repository..."
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Build with Maven') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-21'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                dir("${PROJECT_DIR}") {
                    echo "Building project with Maven..."
                    sh 'mvn clean package -DskipTests'
                    stash includes: 'target/*.jar', name: 'jarFile'
                }
            }
        }

        stage('Build Docker Image') {
            agent any
            steps {
                dir("${PROJECT_DIR}") {
                    echo "Building Docker image..."
                    unstash 'jarFile'
                    sh '''
                        export DOCKER_BUILDKIT=1
                        docker build -t java-api:latest .
                    '''
                }
            }
        }

        stage('Deploy with Docker Compose') {
            agent any
            steps {
                dir("${PROJECT_DIR}") {
                    echo "Creating docker-compose.yml dynamically..."

                    // create docker-compose.yml file dynamically
                    writeFile file: 'docker-compose.yml', text: '''
version: "3.8"

services:
  java-api:
    image: java-api:latest
    container_name: java-api-container
    ports:
      - "8085:8080"
    restart: unless-stopped
'''

                    echo "Starting deployment with Docker Compose..."
                    sh '''
                        docker-compose down || true
                        docker-compose up -d --build
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
        always {
            script {
                node {
                    cleanWs()
                }
            }
        }
    }
}
