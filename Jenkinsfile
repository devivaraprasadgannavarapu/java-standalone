pipeline {
    agent any

    tools {
    maven 'maven'
    jdk 'jdk17'
        }
    
    environment {
        IMAGE_NAME = "dvpking007/jenkins-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git 'https://github.com/devivaraprasadgannavarapu/java-standalone.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

                stage('SonarQube Analysis') { 
            steps { 
                withSonarQubeEnv('sonar-token') { 
                    sh 'mvn sonar:sonar' 
                } 
            } 
        } 

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Docker Push') {
            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Dev') {
            steps {
                sh '''
                docker rm -f dev-container || true

                docker run -d \
                --name dev-container \
                -p 3001:8080 \
                $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Approval For Stage') {
            steps {
                input message: "Deploy to STAGE?"
            }
        }

        stage('Deploy Stage') {
            steps {
                sh '''
                docker rm -f stage-container || true

                docker run -d \
                --name stage-container \
                -p 3002:8080 \
                $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Approval For Prod') {
            steps {
                input message: "Deploy to PROD?"
            }
        }

        stage('Deploy Prod') {
            steps {
                sh '''
                docker rm -f prod-container || true

                docker run -d \
                --name prod-container \
                -p 3003:8080 \
                $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
