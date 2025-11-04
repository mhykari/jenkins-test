pipeline {
    agent { label 'docker-maven-agent' }

    environment {
        IMAGE_NAME = "java-api"
        CONTAINER_NAME = "java-api-container"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                dir('java-api') {
                    sh "ls -l"
                }
            }
        }

        stage('Build with Maven Wrapper (Java 21)') {
            steps {
                dir('java-api') {
                    sh "chmod +x mvnw"
                    sh "./mvnw clean package -DskipTests"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('java-api') {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
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
