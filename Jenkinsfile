pipeline {
    agent none   // We will define agent per stage

    environment {
        IMAGE_NAME = "java-api"
        CONTAINER_NAME = "java-api-container"
    }

    stages {

        stage('Checkout') {
            agent { label 'docker-jdk21' }
            steps {
                checkout scm
                script {
                    // Move into java-api folder
                    dir('java-api') {
                        sh "ls -l"
                    }
                }
            }
        }

        stage('Build with Maven (Java 21)') {
            agent { label 'docker-jdk21' }
            steps {
                dir('java-api') {
                    sh "mvn -version"
                    sh "mvn clean package -DskipTests"
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
                // Copy artifact from workspace to docker build context
                sh """
                cp java-api/target/*.jar .
                docker build -t ${IMAGE_NAME} -f java-api/Dockerfile .
                """
            }
        }

        stage('Stop Old Container & Run New') {
            agent { label 'master' }
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
