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
                sh 'docker buildx version'
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

        stage('Build & Push Images') {
            steps {
                sh 'docker buildx build --platform linux/amd64 -t $DOCKER_USER/vote-app:latest --push ./vote'
                sh 'docker buildx build --platform linux/amd64 -t $DOCKER_USER/result-app:latest --push ./result'
                sh 'docker buildx build --platform linux/amd64 -t $DOCKER_USER/worker-app:latest --push ./worker'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -i /root/.ssh/id_ed25519 -o StrictHostKeyChecking=no ubuntu@54.204.252.248 << EOF
                cd ~/microservices-docker-voting-app
                sudo docker compose pull
                sudo docker compose up -d
                sudo docker image prune -f
                EOF
                '''
            }
        }
    }

    post {
        success {
            echo 'Application built, pushed, and deployed successfully!'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
