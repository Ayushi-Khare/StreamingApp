pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "735381631198"

        FRONTEND_REPO  = "streamingapp-frontend"
        AUTH_REPO      = "streamingapp-auth"
        ADMIN_REPO     = "streamingapp-admin"
        STREAMING_REPO = "streamingapp-streaming"
        CHAT_REPO      = "streamingapp-chat"

        REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    java -version
                    git --version
                    docker --version
                    aws --version
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login \
                    --username AWS \
                    --password-stdin $REGISTRY
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh """
                        docker build \
                        -t ${FRONTEND_REPO}:latest \
                        -f Dockerfile .
                    """
                }
            }
        }

        stage('Build Auth Service') {
            steps {
                dir('backend/authService') {
                    sh """
                        docker build \
                        -t ${AUTH_REPO}:latest \
                        -f Dockerfile .
                    """
                }
            }
        }

        stage('Build Admin Service') {
            steps {
                dir('backend') {
                    sh """
                        docker build \
                        -t ${ADMIN_REPO}:latest \
                        -f adminService/Dockerfile .
                    """
                }
            }
        }

        stage('Build Streaming Service') {
            steps {
                dir('backend') {
                    sh """
                        docker build \
                        -t ${STREAMING_REPO}:latest \
                        -f streamingService/Dockerfile .
                    """
                }
            }
        }

        stage('Build Chat Service') {
            steps {
                dir('backend') {
                    sh """
                        docker build \
                        -t ${CHAT_REPO}:latest \
                        -f chatService/Dockerfile .
                    """
                }
            }
        }

                stage('Tag Images') {
            steps {
                sh """
                    docker tag ${FRONTEND_REPO}:latest ${REGISTRY}/${FRONTEND_REPO}:latest
                    docker tag ${AUTH_REPO}:latest ${REGISTRY}/${AUTH_REPO}:latest
                    docker tag ${ADMIN_REPO}:latest ${REGISTRY}/${ADMIN_REPO}:latest
                    docker tag ${STREAMING_REPO}:latest ${REGISTRY}/${STREAMING_REPO}:latest
                    docker tag ${CHAT_REPO}:latest ${REGISTRY}/${CHAT_REPO}:latest

                    docker tag ${FRONTEND_REPO}:latest ${REGISTRY}/${FRONTEND_REPO}:${BUILD_NUMBER}
                    docker tag ${AUTH_REPO}:latest ${REGISTRY}/${AUTH_REPO}:${BUILD_NUMBER}
                    docker tag ${ADMIN_REPO}:latest ${REGISTRY}/${ADMIN_REPO}:${BUILD_NUMBER}
                    docker tag ${STREAMING_REPO}:latest ${REGISTRY}/${STREAMING_REPO}:${BUILD_NUMBER}
                    docker tag ${CHAT_REPO}:latest ${REGISTRY}/${CHAT_REPO}:${BUILD_NUMBER}
                """
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh """
                    docker push ${REGISTRY}/${FRONTEND_REPO}:latest
                    docker push ${REGISTRY}/${AUTH_REPO}:latest
                    docker push ${REGISTRY}/${ADMIN_REPO}:latest
                    docker push ${REGISTRY}/${STREAMING_REPO}:latest
                    docker push ${REGISTRY}/${CHAT_REPO}:latest

                    docker push ${REGISTRY}/${FRONTEND_REPO}:${BUILD_NUMBER}
                    docker push ${REGISTRY}/${AUTH_REPO}:${BUILD_NUMBER}
                    docker push ${REGISTRY}/${ADMIN_REPO}:${BUILD_NUMBER}
                    docker push ${REGISTRY}/${STREAMING_REPO}:${BUILD_NUMBER}
                    docker push ${REGISTRY}/${CHAT_REPO}:${BUILD_NUMBER}
                """
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                    docker image prune -f
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }

        always {
            cleanWs()
        }
    }
}