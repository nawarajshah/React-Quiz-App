pipeline {
    agent {
        label 'agent1'
    }
    
    tools {
        jdk 'OpenJDK8'
        maven 'Maven3'
    }

    environment {
        DOCKER_REGISTRY = 'nawarajshah/quiz_pp'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = 'latest'
        EC2_INSTANCE_IP = '3.88.22.231'
        EC2_INSTANCE_SSH_USER = 'i-06d0dcfde6f92c549'
        SSH_CREDENTIALS_ID = 'your-ssh-credentials-id'
        DOCKER_CREDENTIALS_ID = 'dockerhubcredential'
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
                    // Authenticate with Docker registry using credentials
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS_ID) {
                        // Build and push Docker image
                        def customImage = docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}", "-f Dockerfile .")
                        customImage.push()
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
