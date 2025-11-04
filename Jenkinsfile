pipeline {
    agent { label 'docker-maven-agent' }

    environment {
        IMAGE_NAME = "java-api"
        CONTAINER_NAME = "java-api-container"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                sh "ls -l java-api"
            }
        }

        stage('Build with Maven (Java 21)') {
            steps {
                dir('java-api') {
                    sh """
                        docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
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
            steps {
                sh """
                cp java-api/target/*.jar .
                docker build -t ${IMAGE_NAME} -f java-api/Dockerfile .
                """
            }
        }

        stage('Stop Old Container & Run New') {
            steps {
                sh """
                if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                fi

                docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${IMAGE_NAME}
                """
            }
        }
    }
}
