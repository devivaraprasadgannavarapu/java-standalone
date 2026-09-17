pipeline {

    agent none

    environment {

        // ==============================
        // AWS / ECR CONFIGURATION
        // ==============================

        AWS_REGION     = 'eu-north-1'
        ECR_REPOSITORY = 'java-app'

        // ==============================
        // Docker / Application
        // ==============================

        IMAGE_NAME     = 'java-standalone'
        CONTAINER_NAME = 'java-standalone'

        CONTAINER_PORT = '8081'
        APP_PORT       = '8080'
    }

    stages {

        // =========================================================
        // STAGE 1: CHECKOUT
        // AGENT 1
        // =========================================================

        stage('Checkout') {

            agent {
                label 'agent-1'
            }

            steps {

                echo '=========================================='
                echo 'CHECKOUT STAGE'
                echo '=========================================='

                checkout scm

                sh '''
                    echo "Running on:"
                    hostname

                    echo ""
                    echo "Workspace:"
                    pwd

                    echo ""
                    echo "Source files:"
                    ls -la
                '''
            }
        }


        // =========================================================
        // STAGE 2: TEST + SONARQUBE
        // AGENT 1
        // =========================================================

        stage('Test') {

            agent {
                label 'agent-1'
            }

            steps {

                echo '=========================================='
                echo 'TEST / SONARQUBE STAGE'
                echo '=========================================='

                sh '''
                    echo "Running on:"
                    hostname

                    echo ""
                    echo "Java version:"
                    java -version

                    echo ""
                    echo "Maven version:"
                    mvn -version
                '''

                withSonarQubeEnv('SonarQube') {

                    withCredentials([
                        string(
                            credentialsId: 'sonarqube-token',
                            variable: 'SONAR_TOKEN'
                        )
                    ]) {

                        sh '''
                            echo "Running Maven tests..."

                            mvn clean test

                            echo ""
                            echo "Running SonarQube analysis..."

                            mvn sonar:sonar \
                                -Dsonar.projectKey=java-standalone \
                                -Dsonar.token="$SONAR_TOKEN"

                            echo ""
                            echo "SonarQube analysis completed."
                        '''
                    }
                }
            }
        }


        // =========================================================
        // STAGE 3: BUILD JAR + DOCKER IMAGE
        // AGENT 2
        // =========================================================

        stage('Build') {

            agent {
                label 'agent-2'
            }

            steps {

                echo '=========================================='
                echo 'BUILD STAGE'
                echo '=========================================='

                // Agent 2 has a separate workspace,
                // therefore checkout source code again.
                checkout scm

                sh '''
                    echo "Running on:"
                    hostname

                    echo ""
                    echo "Java version:"
                    java -version

                    echo ""
                    echo "Maven version:"
                    mvn -version

                    echo ""
                    echo "Docker version:"
                    docker --version

                    echo ""
                    echo "Building JAR..."

                    mvn clean package -DskipTests

                    echo ""
                    echo "Generated JAR:"
                    ls -lh target/

                    echo ""
                    echo "Building Docker image..."

                    docker build \
                        -t "$IMAGE_NAME:$BUILD_NUMBER" \
                        .

                    echo ""
                    echo "Docker images:"
                    docker images | grep "$IMAGE_NAME"
                '''
            }
        }


        // =========================================================
        // STAGE 4: TRIVY SCAN
        // AGENT 2
        // =========================================================

        stage('Scan') {

            agent {
                label 'agent-2'
            }

            steps {

                echo '=========================================='
                echo 'TRIVY SECURITY SCAN'
                echo '=========================================='

                sh '''
                    echo "Running on:"
                    hostname

                    echo ""
                    echo "Scanning image:"
                    echo "$IMAGE_NAME:$BUILD_NUMBER"

                    echo ""
                    echo "Trivy version:"
                    trivy --version

                    echo ""
                    echo "Starting vulnerability scan..."

                    trivy image \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        "$IMAGE_NAME:$BUILD_NUMBER"

                    echo ""
                    echo "Trivy scan completed successfully."
                '''
            }
        }


        // =========================================================
        // STAGE 5: PUSH TO AMAZON ECR
        // AGENT 2
        // =========================================================

        stage('Push to ECR') {

            agent {
                label 'agent-2'
            }

            steps {

                echo '=========================================='
                echo 'PUSH TO AMAZON ECR'
                echo '=========================================='

                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-ecr-credentials'
                    ]
                ]) {

                    sh '''
                        echo "Running on:"
                        hostname

                        echo ""
                        echo "AWS CLI version:"
                        aws --version

                        echo ""
                        echo "AWS Identity:"
                        aws sts get-caller-identity

                        echo ""
                        echo "AWS Region:"
                        echo "$AWS_REGION"

                        echo ""
                        echo "Getting AWS Account ID..."

                        AWS_ACCOUNT_ID=$(aws sts get-caller-identity \
                            --query Account \
                            --output text)

                        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

                        ECR_IMAGE="${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}"

                        echo ""
                        echo "ECR Registry:"
                        echo "$ECR_REGISTRY"

                        echo ""
                        echo "ECR Image:"
                        echo "$ECR_IMAGE"

                        echo ""
                        echo "Logging into Amazon ECR..."

                        aws ecr get-login-password \
                            --region "$AWS_REGION" \
                        | docker login \
                            --username AWS \
                            --password-stdin "$ECR_REGISTRY"

                        echo ""
                        echo "ECR login successful."

                        echo ""
                        echo "Tagging Docker image..."

                        docker tag \
                            "$IMAGE_NAME:$BUILD_NUMBER" \
                            "$ECR_IMAGE"

                        echo ""
                        echo "Docker images:"
                        docker images | grep "$IMAGE_NAME"

                        echo ""
                        echo "Pushing Docker image to ECR..."

                        docker push "$ECR_IMAGE"

                        echo ""
                        echo "=========================================="
                        echo "IMAGE SUCCESSFULLY PUSHED TO ECR"
                        echo "=========================================="

                        echo ""
                        echo "ECR Image:"
                        echo "$ECR_IMAGE"
                    '''
                }
            }
        }


        // =========================================================
        // STAGE 6: DEPLOY ON AGENT-3
        // AGENT 3
        // =========================================================

        stage('Deploy') {

            agent {
                label 'agent-3'
            }

            steps {

                echo '=========================================='
                echo 'DEPLOY STAGE'
                echo '=========================================='

                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-ecr-credentials'
                    ]
                ]) {

                    sh '''
                        echo "Running on:"
                        hostname

                        echo ""
                        echo "AWS CLI version:"
                        aws --version

                        echo ""
                        echo "Docker version:"
                        docker --version

                        echo ""
                        echo "AWS Identity:"
                        aws sts get-caller-identity

                        echo ""
                        echo "Getting AWS Account ID..."

                        AWS_ACCOUNT_ID=$(aws sts get-caller-identity \
                            --query Account \
                            --output text)

                        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

                        ECR_IMAGE="${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}"

                        echo ""
                        echo "ECR Image:"
                        echo "$ECR_IMAGE"

                        echo ""
                        echo "Logging into Amazon ECR..."

                        aws ecr get-login-password \
                            --region "$AWS_REGION" \
                        | docker login \
                            --username AWS \
                            --password-stdin "$ECR_REGISTRY"

                        echo ""
                        echo "ECR login successful."

                        echo ""
                        echo "Pulling image from ECR..."

                        docker pull "$ECR_IMAGE"

                        echo ""
                        echo "Stopping old container..."

                        docker stop "$CONTAINER_NAME" || true

                        echo ""
                        echo "Removing old container..."

                        docker rm "$CONTAINER_NAME" || true

                        echo ""
                        echo "Starting new container..."

                        docker run -d \
                            --name "$CONTAINER_NAME" \
                            -p "$CONTAINER_PORT:$APP_PORT" \
                            "$ECR_IMAGE"

                        echo ""
                        echo "Running containers:"

                        docker ps

                        echo ""
                        echo "=========================================="
                        echo "DEPLOYMENT COMPLETED SUCCESSFULLY"
                        echo "=========================================="
                    '''
                }
            }
        }
    }


    // =============================================================
    // POST ACTIONS
    // =============================================================

    post {

        success {

            echo '''
            ==========================================
            PIPELINE SUCCESS
            ==========================================

            Checkout       : Agent-1
            Test/SonarQube : Agent-1
            Build          : Agent-2
            Trivy Scan     : Agent-2
            ECR Push       : Agent-2
            Deploy         : Agent-3

            ==========================================
            '''
        }

        failure {

            echo '''
            ==========================================
            PIPELINE FAILED
            ==========================================

            Check the console output for the failed stage.

            ==========================================
            '''
        }

        always {

            echo "Pipeline execution completed."
        }
    }
}
