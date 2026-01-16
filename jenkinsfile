pipeline {
    agent any

    environment {
        DOCKER_SERVER_IP = "15.206.187.252"   // second EC2 public IP
        SSH_USER = "root"
        IMAGE_NAME = "jenkins-demo-image"
        CONTAINER_NAME = "jenkins-demo-container"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Copy Files to Docker Server') {
            steps {
                sh """
                scp -o StrictHostKeyChecking=no Dockerfile index.html \
                ${SSH_USER}@${DOCKER_SERVER_IP}:/home/ec2-user/
                """
            }
        }

        stage('Install Docker on Target (if needed)') {
            steps {
                sh """
                ssh ${SSH_USER}@${DOCKER_SERVER_IP} '
                sudo yum install docker -y || true
                sudo systemctl start docker
                sudo systemctl enable docker
                sudo usermod -aG docker ec2-user
                '
                """
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                ssh ${SSH_USER}@${DOCKER_SERVER_IP} '
                docker build -t ${IMAGE_NAME} .
                '
                """
            }
        }

        stage('Docker Run') {
            steps {
                sh """
                ssh ${SSH_USER}@${DOCKER_SERVER_IP} '
                docker rm -f ${CONTAINER_NAME} || true
                docker run -d -p 80:80 --name ${CONTAINER_NAME} ${IMAGE_NAME}
                '
                """
            }
        }
    }

    post {
        success {
            echo "🚀 Docker container running successfully on second EC2!"
        }
    }
}
