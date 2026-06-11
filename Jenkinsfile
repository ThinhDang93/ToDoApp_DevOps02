pipeline {
    agent any

    environment {
        DOCKERHUB_USER    = "thinhdang93"
        BACKEND_IMAGE     = "${DOCKERHUB_USER}/todo-backend-devops02"
        FRONTEND_IMAGE    = "${DOCKERHUB_USER}/todo-frontend-devops02"
        DOCKERHUB_CREDS   = credentials('dockerhub-credentials')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Checkout done"
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
                    echo ${DOCKERHUB_CREDS_PSW} | docker login \
                        -u ${DOCKERHUB_CREDS_USR} \
                        --password-stdin

                    docker push ${BACKEND_IMAGE}:latest
                    docker push ${BACKEND_IMAGE}:${env.BUILD_NUMBER}
                    docker push ${FRONTEND_IMAGE}:latest
                    docker push ${FRONTEND_IMAGE}:${env.BUILD_NUMBER}

                    docker logout
                """
            }
        }

        stage('Deploy Local') {
            steps {
                echo "🚀 Deploying stack locally..."
                sh """
                    cd ${env.WORKSPACE}
                    # Down stack cũ trước (xóa containers, giữ volumes)
                    docker-compose down --remove-orphans
                    # Pull image mới từ DockerHub
                    docker-compose pull mongodb
                    # Khởi động lại toàn bộ stack
                    docker-compose up -d
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
            sh "docker image prune -f"
        }
    }
}
