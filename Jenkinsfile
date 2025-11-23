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

        // 원격 서버 배포 디렉토리 생성
        // stage('Copy to Remote Server') {
        //     steps {
        //         sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
        //             sh """
        //                 ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
        //                 ${REMOTE_USER}@${REMOTE_HOST} "mkdir -p ${REMOTE_DIR}/"
        //             """
        //         }
        //     }
        // }
    }
}
