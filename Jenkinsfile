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

        // Git 최신 소스 코드 가져오기
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        // Git 최신 소스 코드 가져오기
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        // Maven 빌드 및 JAR 생성
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // 빌드 결과물 JAR 파일명을 app.jar 로 변경
        stage('Prepare Jar') {
            steps {
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }

        // 원격 서버 배포 디렉토리 생성 + 파일 업로드
        stage('Copy to Remote Server') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {

                    // 원격 서버 배포 폴더 생성
                    sh """
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${REMOTE_USER}@${REMOTE_HOST} "mkdir -p ${REMOTE_DIR}/"
                    """

                    // JAR + Dockerfile 업로드
                    sh """
                        scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${JAR_FILE_NAME} Dockerfile \
                        ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
                    """
                }
            }
        }

        // 원격 서버 도커 빌드 및 실행
        stage('Remote Docker Build & Deploy') {
            steps {
                sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {

                    sh """
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${REMOTE_USER}@${REMOTE_HOST} << 'ENDSSH'
                            cd ${REMOTE_DIR} || exit 1

                            # stop/remove old container (ignore error)
                            docker rm -f ${CONTAINER_NAME} || true

                            # build docker image (NO CACHE 적용)
                            docker build --no-cache -t ${DOCKER_IMAGE} .

                            # run new container
                            docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}
ENDSSH
                    """
                }
            }
        }
    }
}
