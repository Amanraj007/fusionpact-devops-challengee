pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockers-creds')
        KUBE_CONFIG = credentials('kubeconfig-credentials')
        FRONT_IMAGE = "aman65f/fusionpact-frontend:latest"
        BACK_IMAGE = "aman65f/fusionpact-backend:latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '🔹 Checking out code from Git...'
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                echo '🔹 Building Docker images...'
                sh '''
                    docker build -t $FRONT_IMAGE ./frontend
                    docker build -t $BACK_IMAGE ./backend
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                echo '🔹 Pushing Docker images to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockers-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "aman65f" --password-stdin
                        docker push $FRONT_IMAGE
                        docker push $BACK_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🔹 Deploying to Kubernetes...'
                script {
                    writeFile file: 'config', text: "${KUBE_CONFIG}"
                    sh '''
                        mkdir -p ~/.kube
                        cp config ~/.kube/config

                        kubectl apply -f k8s/
                        kubectl rollout restart deployment fusionpact-frontend
                        kubectl rollout restart deployment fusionpact-backend
                        kubectl rollout status deployment/fusionpact-frontend
                        kubectl rollout status deployment/fusionpact-backend
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful! Application is live on Kubernetes.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}

