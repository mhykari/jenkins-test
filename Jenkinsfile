pipeline {
    agent none

    environment {
        REPO_URL = 'https://github.com/mhykari/jenkins-test.git'
        PROJECT_DIR = 'java-api'
        IMAGE_NAME = 'java-api'
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
                    image 'maven:3.9.6-eclipse-temurin-21'
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
                    echo "Building Docker image on controller host..."
                    unstash 'jarFile'
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Deploy with Docker Compose') {
            agent any
            steps {
                dir("${PROJECT_DIR}") {
                    echo "Deploying application using Docker Compose..."
                    sh """
                        docker compose down || true
                        docker compose up -d --build
                    """
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
