pipeline {
    agent none

    environment {
        IMAGE_NAME = "java-api"
        CONTAINER_NAME = "java-api-container"
    }

    stages {
        stage('Checkout Code') {
            agent { label 'docker-maven-agent' }
            steps {
                checkout scm
                dir('java-api') {
                    sh "ls -l"
                }
            }
        }

        stage('Build with Maven (Java 21)') {
            agent { label 'docker-maven-agent' }
            steps {
                dir('java-api') {
                    sh """
                        docker run --rm \
                        -v \$PWD:/app -w /app \
                        -v \$HOME/.m2:/root/.m2 \
                        maven:3.9.6-eclipse-temurin-21 \
                        mvn clean package -DskipTests
                    """
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'java-api/target/*.jar', fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            agent { label 'master' }
            steps {
                sh """
                cp java-api/target/*.jar .
                docker build -t ${IMAGE_NAME} -f java-api/Dockerfile .
