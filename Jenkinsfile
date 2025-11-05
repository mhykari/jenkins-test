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
                echo "🔄 Cloning repository..."
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Build with Maven (Agent: mvn)') {
            agent { label 'mvn' }
            steps {
                dir("${PROJECT_DIR}") {
                    echo "⚙️ Building project with Maven..."
                    sh 'mvn clean package -DskipTests'
                }
            }
            post {
                success {
                    echo "✅ Maven build complete. Archiving artifact..."
                    archiveArtifacts artifacts: "${PROJECT_DIR}/target/*.jar", fingerprint: true
                }
            }
        }

        stage('Prepare Artifact (Controller)') {
            agent { label 'master' }
            steps {
                echo "📦 Copying built JAR to controller workspace..."
                dir("${PROJECT_DIR}/target") {
                    // Extract artifact from Jenkins storage
                    unstash name: 'workspace'
                }
            }
        }

        stage('Build Docker Image (Controller)') {
            agent { label 'master' }
            steps {
                dir("${PROJECT_DIR}") {
                    echo "🐳 Building Docker image on controller host..."
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Deploy with Docker Compose (Controller)') {
            agent { label 'master' }
            steps {
                dir("${PROJECT_DIR}") {
                    echo "🚀 Deploying applicati
