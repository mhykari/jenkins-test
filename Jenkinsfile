pipeline {
    agent none

    environment {
        REPO_URL = 'https://github.com/mhykari/jenkins-test.git'
        PROJECT_DIR = 'java-api'
        IMAGE_NAME = 'java-api'
        CONTAINER_NAME = 'java-api-container'
        COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {
        stage('Checkout Code') {
            agent any
            steps {
                echo "Cloning repository..."
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Build with Maven (Agent: mvn)') {
            agent { label 'mvn' }
            steps {
                dir("${PROJECT_DIR}") {
                    echo "Building project with Maven..."
                    sh 'mvn clean package -DskipTests'
                    stash includes: 'target/*.jar', name: 'jarFile'
                }
            }
        }

        stage('Build Docker Image (Controller)') {
            agent { label 'master' }
            steps {
                dir("${PROJECT_DIR}") {
                    echo "Building Docker image on controller host..."
                    unstash 'jarFile'
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Deploy with Docker Compose (Controller)') {
            agent { label 'master' }
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
            cleanWs()
        }
    }
}
