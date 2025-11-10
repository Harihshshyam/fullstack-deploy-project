pipeline {
    agent any

    environment {
        SERVER_IP = "15.207.30.60"
        DOCKER_HUB_REPO = "sagar592"   // Replace with your DockerHub username
        FRONTEND_IMAGE = "project_frontend"
        BACKEND_IMAGE = "project_backend"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Harihshshyam/fullstack-deploy-project.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "🔹 Building Backend Docker image..."
                sh "docker build -t $DOCKER_HUB_REPO/$BACKEND_IMAGE:latest ./backend"

                echo "🔹 Building Frontend Docker image..."
                sh "docker build -t $DOCKER_HUB_REPO/$FRONTEND_IMAGE:latest ./frontend"
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push $DOCKER_HUB_REPO/$BACKEND_IMAGE:latest
                        docker push $DOCKER_HUB_REPO/$FRONTEND_IMAGE:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy on EC2 Server') {
            steps {
                sshagent(['server-key']) {
                    sh '''
                        echo "🚀 Deploying on EC2 Server..."
                        scp -o StrictHostKeyChecking=no docker-compose.yaml ubuntu@$SERVER_IP:/home/ubuntu/
                        ssh -o StrictHostKeyChecking=no ubuntu@$SERVER_IP "
                            cd /home/ubuntu &&
                            sudo docker-compose down &&
                            sudo docker-compose pull &&
                            sudo docker-compose up -d
                        "
                        echo "✅ Deployment Completed Successfully!"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Deployment Failed. Please check logs."
        }
    }
}
