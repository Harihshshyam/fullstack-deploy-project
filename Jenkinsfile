pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = credentials('dockerhub-username')  // Jenkins credential ID
        DOCKER_HUB_PASS = credentials('dockerhub-password')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Harihshshyam/fullstack-deploy-project.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t ${DOCKER_HUB_USER}/backend:latest ./backend'
                sh 'docker build -t ${DOCKER_HUB_USER}/frontend:latest ./frontend'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push $USER/backend:latest'
                    sh 'docker push $USER/frontend:latest'
                }
            }
        }

        stage('Deploy on Server') {
            steps {
                sshagent(['server-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@<YOUR_SERVER_IP> "
                        cd /home/ubuntu/project &&
                        docker-compose pull &&
                        docker-compose down &&
                        docker-compose up -d
                    "
                    '''
                }
            }
        }
    }
}
