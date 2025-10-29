pipeline {
    agent any
    tools {
        maven 'maven'    
    }

    environment {
        DOCKER_IMAGE = "shyam0115/webapp"
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
        CONTAINER_NAME = "webapp_container"
        DOCKER_CREDENTIALS = credentials('admin') // Jenkins credentials ID
    }

    stages {

        stage('Build WAR') {
            steps {
                echo "Building WAR file..."
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    echo 'WAR build successful '
                }
                failure {
                    echo 'WAR build failed '
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "sudo docker build -t ${DOCKER_IMAGE}:latest ."
            }
            post {
                success {
                    echo 'Docker image built successfully '
                }
                failure {
                    echo 'Docker image build failed '
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                echo "Tagging Docker image with ${IMAGE_TAG}..."
                sh "sudo docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${IMAGE_TAG}"
            }
            post {
                success {
                    echo "Image tagged successfully "
                }
                failure {
                    echo "Image tagging failed "
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                echo "Logging in to DockerHub..."
                sh 'sudo echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin'
                echo "Pushing image to DockerHub..."
                sh "sudo docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                sh "sudo docker push ${DOCKER_IMAGE}:latest"
            }
            post {
                success {
                    echo "Image pushed to DockerHub "
                }
                failure {
                    echo "Image push failed "
                }
            }
        }

        stage('Clean up Local Images') {
            steps {
                echo "Removing local Docker images..."
                sh "sudo docker rmi ${DOCKER_IMAGE}:${IMAGE_TAG} || true"
                sh "sudo docker rmi ${DOCKER_IMAGE}:latest || true"
            }
            post {
                success {
                    echo "Local images removed "
                }
                failure {
                    echo "Failed to remove local images "
                }
            }
        }

        stage('Run Container') {
            steps {
                echo "Running Docker container..."
                sh "sudo docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${DOCKER_IMAGE}:${IMAGE_TAG}"
            }
            post {
                success {
                    echo "Container running successfully "
                }
                failure {
                    echo "Container failed to start "
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished — cleaning workspace "
            cleanWs()
        }
        success {
            echo "Pipeline executed successfully "
        }
        failure {
            echo "Pipeline failed. Check logs for details "
        }
    }
}
