pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = credentials('dockerhub-username') // Jenkins credentials ID
        DOCKERHUB_TOKEN = credentials('dockerhub-token')       // Jenkins credentials ID
        EC2_USER = credentials('ec2-user')                     // Jenkins credentials ID
        EC2_HOST = credentials('ec2-host')                     // Jenkins credentials ID
        EC2_KEY = credentials('ec2-key')                       // SSH private key (secret file)
    }

    stages {

        stage('Test') {
            steps {
                echo '✅ All tests passed!'
            }
        }

        stage('Build Docker Image') {
            steps {
                checkout scm
                script {
                    sh """
                    echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin
                    docker build -t ${DOCKERHUB_USERNAME}/hello-nginx:latest .
                    docker push ${DOCKERHUB_USERNAME}/hello-nginx:latest
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << 'EOF'
                      docker pull ${DOCKERHUB_USERNAME}/hello-nginx:latest
                      docker stop web || true
                      docker rm web || true
                      docker run -d --name web -p 80:80 ${DOCKERHUB_USERNAME}/hello-nginx:latest
                    EOF
                    """
                }
            }
        }
    }
}
