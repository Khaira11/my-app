pipeline {
    agent any

    environment {
        // DockerHub credentials ID stored in Jenkins Credentials
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'

        // Your DockerHub username
        DOCKER_USER = 'khaira23'

        // App name and version (you can update version for every release)
        APP_NAME = 'my-app'
        APP_VERSION = 'v1.0'

        // Construct full image name dynamically
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
                sh '''
                    export DOCKER_CLIENT_TIMEOUT=300
                    export COMPOSE_HTTP_TIMEOUT=300
                    export DOCKER_BUILDKIT=1
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage('Login & Push to DockerHub') {
            steps {
                echo "📦 Pushing ${IMAGE_NAME} to DockerHub"
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying ${IMAGE_NAME} to Kubernetes cluster"
                sh '''
                    kubectl set image deployment/my-app-deployment my-app-container=${IMAGE_NAME} --record || true
                    kubectl rollout status deployment/my-app-deployment || true
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "🔍 Verifying deployment status..."
                sh '''
  pipeline {
    agent any

    environment {
        // DockerHub credentials ID stored in Jenkins Credentials
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'

        // Your DockerHub username
        DOCKER_USER = 'khaira23'

        // App name and version (you can update version for every release)
        APP_NAME = 'my-app'
        APP_VERSION = 'v1.0'

        // Construct full image name dynamically
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
                sh '''
                    export DOCKER_CLIENT_TIMEOUT=300
                    export COMPOSE_HTTP_TIMEOUT=300
                    export DOCKER_BUILDKIT=1
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage('Login & Push to DockerHub') {
            steps {
                echo "📦 Pushing ${IMAGE_NAME} to DockerHub"
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying ${IMAGE_NAME} to Kubernetes cluster"
                sh '''
                    kubectl set image deployment/my-app-deployment my-app-container=${IMAGE_NAME} --record || true
                    kubectl rollout status deployment/my-app-deployment || true
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "🔍 Verifying deployment status..."
                sh '''
                    kubectl get pods -o wide
                    kubectl get svc
                '''
            }
        }
    }

    post {
        always {
            echo "🎉 Pipeline execution completed"
        }
        failure {
            echo "❌ Pipeline failed — please check the logs above."
        }
    }
}
                    kubectl get svc
                '''
            }
        }
    }

    post {
        always {
            echo "🎉 Pipeline execution completed"
        }
        failure {
            echo "❌ Pipeline failed — please check the logs above."
        }
    }
}

