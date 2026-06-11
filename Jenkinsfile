pipeline {
    // Chạy trên bất kỳ agent nào available
    agent any

    // Khai báo biến dùng chung toàn pipeline
    environment {
        DOCKERHUB_USER    = "thinhdang93"
        BACKEND_IMAGE     = "${DOCKERHUB_USER}/todo-backend-devops02"
        FRONTEND_IMAGE    = "${DOCKERHUB_USER}/todo-frontend-devops02"
        // Lấy credentials đã lưu trong Jenkins
        DOCKERHUB_CREDS   = credentials('dockerhub-credentials')
    }

    stages {

        stage('Checkout') {
            steps {
                // Pull code mới nhất từ GitHub
                checkout scm
                echo "✅ Checkout done — branch: ${env.BRANCH_NAME}"
            }
        }

        stage('Build Backend') {
            steps {
                echo "🔨 Building backend image..."
                sh """
                    docker build -t ${BACKEND_IMAGE}:latest \
                                 -t ${BACKEND_IMAGE}:${env.BUILD_NUMBER} \
                                 ./backend
                """
                // BUILD_NUMBER: số thứ tự build Jenkins tự tăng
                // Dùng để tag image theo build number thay vì commit SHA
            }
        }

        stage('Build Frontend') {
            steps {
                echo "🔨 Building frontend image..."
                sh """
                    docker build -t ${FRONTEND_IMAGE}:latest \
                                 -t ${FRONTEND_IMAGE}:${env.BUILD_NUMBER} \
                                 ./frontend
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "📤 Pushing images to DockerHub..."
                sh """
                    # Login dùng token (không dùng password)
                    echo ${DOCKERHUB_CREDS_PSW} | docker login \
                        -u ${DOCKERHUB_CREDS_USR} \
                        --password-stdin

                    docker push ${BACKEND_IMAGE}:latest
                    docker push ${BACKEND_IMAGE}:${env.BUILD_NUMBER}
                    docker push ${FRONTEND_IMAGE}:latest
                    docker push ${FRONTEND_IMAGE}:${env.BUILD_NUMBER}

                    # Logout sau khi push xong (bảo mật)
                    docker logout
                """
            }
        }

        stage('Deploy Local') {
            steps {
                echo "🚀 Deploying stack locally..."
                sh """
                    cd ${env.WORKSPACE}
                    # Pull image mới nhất từ DockerHub
                    docker compose pull
                    # Restart stack với image mới
                    docker compose up -d --force-recreate
                """
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline SUCCESS — Build #${env.BUILD_NUMBER}"
        }
        failure {
            echo "❌ Pipeline FAILED — Build #${env.BUILD_NUMBER}"
        }
        always {
            // Xóa image local sau khi push để tiết kiệm disk
            sh """
                docker image prune -f
            """
        }
    }
}
