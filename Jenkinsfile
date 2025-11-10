pipeline {
    agent any

    environment {
        SERVER_IP = "15.207.30.60"
        IMAGE_NAME = "myapp"
        DOCKER_USER = "sagar592"   // apna DockerHub username
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/harihshshyam/fullstack-deploy-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "🔨 Building Docker Image..."
                docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                    echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                    docker tag $IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:latest
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
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
                    ssh -o StrictHostKeyChecking=no ubuntu@$SERVER_IP '
                        sudo docker pull $DOCKER_USER/$IMAGE_NAME:latest &&
                        sudo docker stop myapp || true &&
                        sudo docker rm myapp || true &&
                        sudo docker run -d --name myapp -p 80:80 $DOCKER_USER/$IMAGE_NAME:latest
                    '
                    echo "✅ Deployment Completed Successfully!"
                    '''
                }
            }
        }
    }
}
