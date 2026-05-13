pipeline {

    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        IMAGE_NAME = "261142222215.dkr.ecr.eu-north-1.amazonaws.com/java-standalone"
        AWS_REGION = "eu-north-1"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/devivaraprasadgannavarapu/java-standalone.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t java-app .'
            }
        }

        stage('Push to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin 261142222215.dkr.ecr.eu-north-1.amazonaws.com

                docker tag java-app:latest $IMAGE_NAME:latest

                docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy Docker Container') {
            steps {
                sh '''
                docker stop java-container || true
                docker rm java-container || true

                docker run -d \
                --name java-container \
                -p 8081:8080 \
                $IMAGE_NAME:latest
                '''
            }
        }
    }
}
