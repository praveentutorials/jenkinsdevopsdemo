pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "sripraveen/flask-jenkins-demo"
        DOCKER_TAG   = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from Git...'

                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                echo 'Checking Windows/Jenkins environment...'

                bat 'docker --version'
                bat 'kubectl version --client'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."

                bat """
                    docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% .
                """
            }
        }

        stage('Docker Login') {
            steps {
                echo 'Logging in to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    bat '''
                        echo %DOCKER_PASSWORD% | docker login ^
                            -u %DOCKER_USERNAME% ^
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image to Docker Hub..."

                bat """
                    docker push %DOCKER_IMAGE%:%DOCKER_TAG%
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying Kubernetes manifests...'

                bat """
                    kubectl apply -f k8s\\deployment.yaml
                    kubectl apply -f k8s\\service.yaml
                """
            }
        }

        stage('Update Kubernetes Image') {
            steps {
                echo 'Updating Kubernetes deployment image...'

                bat """
                    kubectl set image deployment/flask-app ^
                        flask-app=%DOCKER_IMAGE%:%DOCKER_TAG%
                """
            }
        }

        stage('Wait for Deployment') {
            steps {
                echo 'Waiting for Kubernetes rollout...'

                bat """
                    kubectl rollout status deployment/flask-app --timeout=120s
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Checking Kubernetes resources...'

                bat """
                    echo ==============================
                    echo DEPLOYMENTS
                    echo ==============================
                    kubectl get deployments

                    echo.
                    echo ==============================
                    echo PODS
                    echo ==============================
                    kubectl get pods

                    echo.
                    echo ==============================
                    echo SERVICES
                    echo ==============================
                    kubectl get services
                """
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo "Deployment completed successfully!"
            echo "Docker Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "Application: http://localhost:30080"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "Pipeline failed!"
            echo "Check the Jenkins Console Output."
            echo "======================================"
        }

        always {
            bat 'docker logout || exit /b 0'
        }
    }
}