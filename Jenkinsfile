pipeline {
    agent any

    environment {
        IMAGE_NAME = "jaava-api"
        CONTAINER_NAME = "java-api-container"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mhykari/jenkins-test/'
            }
        }

        stage('Build with Maven') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Stop Previous Container') {
            steps {
                sh """
                if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                    echo "Stopping old container..."
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                fi
                """
            }
        }

        stage('Run New Container') {
            steps {
                sh "docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${IMAGE_NAME}"
            }
        }
    }
}
