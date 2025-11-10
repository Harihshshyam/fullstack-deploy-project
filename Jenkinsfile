pipeline {
    agent any

    environment {
        DOCKERHUB_USER = credentials('dockerhub-username')
        DOCKERHUB_PASS = credentials('dockerhub-password')
        DEPLOY_SERVER = '15.207.30.60'      // <-- yahan apna EC2 IP likho
        IMAGE_NAME = 'harishyam/fullstack'  // <-- yahan apna DockerHub repo name likho
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '🔹 Cloning the repository...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🔹 Building Docker image...'
                sh '''
                    docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo '🔹 Pushing Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy on EC2 Server') {
            steps {
                echo '🔹 Deploying container on EC2...'
                sshagent(['server-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$DEPLOY_SERVER '
                            docker stop fullstack || true &&
                            docker rm fullstack || true &&
                            docker pull $IMAGE_NAME:latest &&
                            docker run -d --name fullstack -p 80:80 $IMAGE_NAME:latest
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed. Please check logs.'
        }
    }
}
