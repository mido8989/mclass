pipeline {
    agent any

    tools {
        maven 'maven 3.9.11'
    }

    environment {
        // 배포 관련 환경 변수 설정
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

        // Git 최신 소스 코드 체크아웃
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        // Maven 빌드 (테스트 생략)
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // JAR 파일 이름 통일 (app.jar)
        stage('Prepare Jar') {
            steps {
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }

        // 원격 서버 배포 디렉토리 생성 및 파일 업로드
        stage('Copy to Remote Server') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {

                    // 배포 디렉토리 생성
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

        // 원격 서버 도커 빌드 및 컨테이너 실행
        stage('Remote Docker Build & Deploy') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {

                    sh '''
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} << "ENDSSH"

cd ${REMOTE_DIR} || exit 1

# 기존 컨테이너 종료 및 삭제
docker rm -f ${CONTAINER_NAME} || true

# 도커 이미지 빌드 (no-cache)
docker build --no-cache -t ${DOCKER_IMAGE} .

# 새 컨테이너 실행
docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}

ENDSSH
'''
                }
            }
        }
    }
}
