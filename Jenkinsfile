pipeline {
    agent any

    tools {
        maven 'maven 3.9.11'
    }

    environment {
        // ===== 배포 환경 변수 =====
        DOCKER_IMAGE = "demo-app"              // Docker 이미지 이름
        CONTAINER_NAME = "springboot-container" // Docker 컨테이너 이름
        JAR_FILE_NAME = "app.jar"               // 빌드 결과물 이름
        PORT = "8081"                           // EC2에서 리스닝할 포트

        REMOTE_USER = "ec2-user"                // EC2 유저
        REMOTE_HOST = "3.34.81.134"             // EC2 Public IP
        REMOTE_DIR = "/home/ec2-user/deploy"    // 배포 경로

        SSH_CREDENTIALS_ID = "4a18bd24-2557-4926-b3c7-28b9e363f21b" // SSH Key ID
    }

    stages {

        // ===== Git Checkout =====
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        // ===== Maven Build =====
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // ===== JAR 리네이밍 =====
        stage('Prepare Jar') {
            steps {
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }

        // ===== EC2 서버에 파일 복사 =====
        stage('Copy to Remote Server') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {

                    // 원격 디렉토리 생성
                    sh """
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${REMOTE_USER}@${REMOTE_HOST} "mkdir -p ${REMOTE_DIR}/"
                    """

                    // JAR + Dockerfile 전송
                    sh """
                        scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${JAR_FILE_NAME} Dockerfile \
                        ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
                    """
                }
            }
        }

        // ===== EC2 도커 빌드 & 배포 =====
        stage('Remote Docker Build & Deploy') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {

                    sh """
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} << 'ENDSSH'
cd ${REMOTE_DIR} || exit 1

echo ">>> 기존 컨테이너 종료 중..."
docker rm -f ${CONTAINER_NAME} || true

echo ">>> Docker 이미지 빌드 중..."
docker build --no-cache -t ${DOCKER_IMAGE} .

echo ">>> 새 컨테이너 실행 중..."
docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}

echo ">>> 배포 완료!"
ENDSSH
                    """
                }
            }
        }
    }
}
