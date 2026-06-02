pipeline {
    agent any

    environment {
        // Update these values
        DOCKER_IMAGE_NAME = "your-dockerhub-username/cicd-demo-app"
        DOCKER_HUB_CREDS  = "docker-hub-credentials-id"
        K8S_CREDS         = "k8s-config-credentials-id"
        IMAGE_TAG         = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS}", passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        echo "Logging into Docker Hub..."
                        sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                        echo "Pushing image..."
                        sh "docker push ${DOCKER_IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes..."
                    withKubeConfig([credentialsId: "${K8S_CREDS}"]) {
                        // Replace placeholder with actual image name
                        sh "sed -i 's|DOCKER_IMAGE_PLACEHOLDER|${DOCKER_IMAGE_NAME}:${IMAGE_TAG}|g' k8s-deployment.yaml"
                        sh "kubectl apply -f k8s-deployment.yaml"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up..."
            sh "docker logout"
        }
    }
}
