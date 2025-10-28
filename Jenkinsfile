pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/khaira23"         // Your Docker Hub username
        IMAGE_NAME = "my-python-app"
        GIT_BRANCH = "main"
        KUBE_DEPLOYMENT = "python-app-deployment"
        KUBE_NAMESPACE = "default"
    }

    triggers {
        githubPush()  // Automatically triggers when code is pushed to GitHub
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: 'https://github.com/Khaira11/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '☸️ Deploying application to Kubernetes...'
                withKubeConfig([credentialsId: 'k8s-credential']) {
                    sh '''
                        # Apply Deployment and Service files
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml

                        # Optional: Wait for rollout to complete
                        kubectl rollout status deployment/$KUBE_DEPLOYMENT -n $KUBE_NAMESPACE
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed."
        }
    }
}

