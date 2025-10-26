pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        DOCKER_USER = 'khaira23'
        APP_NAME = 'my-app'
        APP_VERSION = 'v1.0'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}:${APP_VERSION}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code from GitHub"
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🔨 Building Docker image: ${IMAGE_NAME}"
                sh """
                    export DOCKER_CLIENT_TIMEOUT=300
                    export COMPOSE_HTTP_TIMEOUT=300
                    export DOCKER_BUILDKIT=1
                    docker build -t ${IMAGE_NAME} .
                """
            }
        }

        stage('Login & Push to DockerHub') {
            steps {
                echo "📦 Logging in and pushing image to DockerHub"
                withCredentials([usernamePassword(credentialsId: "${docker-hub-credentials}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying ${IMAGE_NAME} to Kubernetes"
                sh """
                    kubectl set image deployment/my-app-deployment my-app-container=${IMAGE_NAME} --record || true
                    kubectl rollout status deployment/my-app-deployment || true
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "🔍 Checking Kubernetes resources..."
                sh """
                    kubectl get pods -o wide
                    kubectl get svc
                """
            }
        }
    }

    post {
        always {
            echo "🎉 Pipeline execution completed"
        }
        failure {
            echo "❌ Pipeline failed — check logs above."
        }
    }
}

