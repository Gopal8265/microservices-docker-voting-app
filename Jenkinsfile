pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
        DOCKER_USER = 'gopal82'
        IMAGE_NAME = 'hospital-mis'
        IMAGE_TAG = 'v1'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "===== PATH ====="
                    echo $PATH

                    echo "===== Git ====="
                    which git
                    git --version

                    echo "===== Docker ====="
                    which docker
                    docker --version
                    docker ps

                    echo "===== Kubectl ====="
                    which kubectl
                    kubectl version --client

                    echo "===== Minikube ====="
                    minikube status
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                    kubectl apply -f k8s/
                    kubectl rollout restart deployment/hospital-app
                    kubectl rollout status deployment/hospital-app
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            sh 'docker logout || true'
        }
    }
}