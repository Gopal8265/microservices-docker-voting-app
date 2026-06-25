pipeline {
    agent any

    environment {
        DOCKER_USER = 'gopal82'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Docker') {
            steps {
                sh 'docker --version'
                sh 'docker ps'
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

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $DOCKER_USER/vote-app:latest ./vote'
                sh 'docker build -t $DOCKER_USER/result-app:latest ./result'
                sh 'docker build -t $DOCKER_USER/worker-app:latest ./worker'
            }
        }

        stage('Push Images') {
            steps {
                sh 'docker push $DOCKER_USER/vote-app:latest'
                sh 'docker push $DOCKER_USER/result-app:latest'
                sh 'docker push $DOCKER_USER/worker-app:latest'
            }
        }
    }

    post {
        success {
            echo 'Docker images built and pushed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}