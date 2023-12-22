pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'nawarajshah/quiz_pp'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = 'latest'
        EC2_INSTANCE_IP = '3.95.134.75'
        EC2_INSTANCE_SSH_USER = 'ec2-user'
        SSH_CREDENTIALS_ID = 'your-ssh-credentials-id'
        REMOTE_DOCKER_COMPOSE_FILE = 'path/to/your/docker-compose.yml'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}", "-f Dockerfile .")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://${DOCKER_REGISTRY}', 'docker-credentials-id') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                script {
                    sshCommand remote: [
                        host: EC2_INSTANCE_IP,
                        user: EC2_INSTANCE_SSH_USER,
                        port: 22,
                        identityFile: [$class: 'FileParameterValue', name: 'SSH_KEY', file: 'path/to/your/private-key.pem']
                    ], command: """
                        docker-compose -f ${REMOTE_DOCKER_COMPOSE_FILE} pull
                        docker-compose -f ${REMOTE_DOCKER_COMPOSE_FILE} up -d
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image build, push, and deploy succeeded!'
        }
        failure {
            echo 'Docker image build, push, and deploy failed!'
        }
    }
}
