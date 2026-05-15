pipeline {
    agent any

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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=demo-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=YOUR_SONAR_TOKEN
                    '''
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
                -p 3001:3000 \
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
                -p 3002:3000 \
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
                -p 3003:3000 \
                $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
