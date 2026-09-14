pipeline {

    agent any

    environment {
        AWS_REGION = "eu-north-1"
        ECR_REGISTRY = "540553462149.dkr.ecr.eu-north-1.amazonaws.com"
        IMAGE_NAME = "540553462149.dkr.ecr.eu-north-1.amazonaws.com/my-app"
        CONTAINER_NAME = "java-container"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/devivaraprasadgannavarapu/java-standalone.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t java-app:latest .
                '''
            }
        }

        stage('Push to Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login \
                    --username AWS \
                    --password-stdin $ECR_REGISTRY

                    docker tag java-app:latest $IMAGE_NAME:latest

                    docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy Docker Container') {
            steps {
                sh '''
                    docker pull $IMAGE_NAME:latest

                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true

                    docker run -d \
                        --name $CONTAINER_NAME \
                        -p 8081:8080 \
                        $IMAGE_NAME:latest
                '''
            }
        }
    }
}
