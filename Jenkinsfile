pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        IMAGE_NAME = "khaira23/my-python-app"
        IMAGE_TAG = "build-${BUILD_NUMBER}"  // unique tag per build
        DEPLOYMENT_FILE = "k8s/deployment.yaml"
        KUBE_DEPLOYMENT = "flask-deployment"
        KUBE_NAMESPACE = "default"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Khaira11/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "🏗️ Building Docker image..."
                docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'Docker_hub', variable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u khaira23 --password-stdin ${REGISTRY}
                    docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker logout ${REGISTRY}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'k8s-credential']) {
                    sh '''
                    echo "🚀 Deploying application to Kubernetes..."
                    
                    # Replace the image placeholder dynamically
                    sed -i "s|IMAGE_PLACEHOLDER|${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}|g" ${DEPLOYMENT_FILE}

                    # Apply deployment
                    kubectl apply -f ${DEPLOYMENT_FILE}
                    kubectl rollout status deployment/${KUBE_DEPLOYMENT} -n ${KUBE_NAMESPACE}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed. Build number: ${BUILD_NUMBER}"
        }
    }
}

