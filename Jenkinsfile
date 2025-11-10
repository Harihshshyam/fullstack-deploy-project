pipeline {
    agent any

    environment {
        DOCKERHUB_USER = credentials('dockerhub-username') // Jenkins Credentials ID for DockerHub username
        DOCKERHUB_PASS = credentials('dockerhub-password') // Jenkins Credentials ID for DockerHub password
        SERVER_IP     = "15.207.30.60"
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo "🔹 Cloning repository..."
                git branch: 'main', url: 'https://github.com/Harihshshyam/fullstack-deploy-project.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "🔹 Building Docker images..."
                sh 'docker build -t mybackend:latest ./backend'
                sh 'docker build -t myfrontend:latest ./frontend'
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "🔹 Pushing Docker images to DockerHub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker tag mybackend:latest $DOCKERHUB_USER/backend:latest
                        docker tag myfrontend:latest $DOCKERHUB_USER/frontend:latest
                        docker push $DOCKERHUB_USER/backend:latest
                        docker push $DOCKERHUB_USER/frontend:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy on EC2 Server') {
            steps {
                echo "🚀 Deploying on EC2 Server..."
                sshagent(['ubuntu-ec2-key']) {  // Jenkins Credentials ID for EC2 private key
                    sh """
                        scp -o StrictHostKeyChecking=no docker-compose.yaml ubuntu@$SERVER_IP:/home/ubuntu/
                        ssh -o StrictHostKeyChecking=no ubuntu@$SERVER_IP '
                            echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin &&
                            cd /home/ubuntu &&
                            sudo docker-compose down &&
                            sudo docker-compose pull &&
                            sudo docker-compose up -d &&
                            docker logout
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Completed Successfully!"
        }
        failure {
            echo "❌ Deployment Failed. Please check logs."
        }
    }
}
