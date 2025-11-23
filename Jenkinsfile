pipeline {
    agent any

    tools {
        maven 'maven 3.9.11'
    }

    environment {
        DOCKER_IMAGE = "demo-app"
        CONTAINER_NAME = "springboot-container"
        JAR_FILE_NAME = "app.jar"
        PORT = "8081"

        REMOTE_USER = "ec2-user"
        REMOTE_HOST = "3.34.81.134"
        REMOTE_DIR = "/home/ec2-user/deploy"

        SSH_CREDENTIALS_ID = "4a18bd24-2557-4926-b3c7-28b9e363f21b"
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Prepare Jar') {
            steps {
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }

        stage('Copy to Remote Server') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {

                    // Create remote directory
                    sh """
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${REMOTE_USER}@${REMOTE_HOST} "mkdir -p ${REMOTE_DIR}/"
                    """

                    // Copy JAR + Dockerfile
                    sh """
                        scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${JAR_FILE_NAME} Dockerfile \
                        ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
                    """
                }
            }
        }

        stage('Remote Docker Build & Deploy') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${REMOTE_USER}@${REMOTE_HOST} << 'ENDSSH'

                            cd ${REMOTE_DIR} || exit 1

                            echo ">>> Stopping old container (if exists)..."
                            docker rm -f ${CONTAINER_NAME} || true

                            echo ">>> Building Docker image (NO CACHE)..."
                            docker build --no-cache -t ${DOCKER_IMAGE} .

                            echo ">>> Running new container..."
                            docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}

                        ENDSSH
                    """
                }
            }
        }
    }
}
