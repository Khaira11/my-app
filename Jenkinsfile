pipeline {
    agent any

    environment {
        DOCKER_REPO = 'khaira23/flask-jenkins'
        DEPLOYMENT_NAME = 'flask-deployment'
        NAMESPACE = 'default'
    }

    stages {

        stage('Set Version Tag') {
            steps {
                script {
                    // Properly declare variables
                    def version = "v${BUILD_NUMBER}"
                    def imageName = "${DOCKER_REPO}:${version}"

                    // Export them for next stages
                    env.VERSION = version
                    env.IMAGE_NAME = imageName

                    echo "🧾 New Image Version: ${env.IMAGE_NAME}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🔨 Building Docker image...'
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Login & Push to DockerHub') {
            steps {
                echo '🔐 Logging in to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }


 stage('Deploy to Kubernetes') {
            steps {
                echo '☸️ Deploying application to Kubernetes...'
                withKubeConfig([credentialsId: 'k8s-credential']) {

echo "🔧 Preparing deployment file..."
                sed "s|IMAGE_PLACEHOLDER|${IMAGE_NAME}|g" k8s/deployment.yaml > k8s/deployment-temp.yaml

                    sh '''
                        # Apply Deployment and Service files
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml

                        # Optional: Wait for rollout to complete
                        kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE
                    '''
                }
            }
        }


        stage('Update Kubernetes Deployment') {
            steps {
                echo '☸️ Updating deployment in Kubernetes...'
                withKubeConfig([credentialsId: 'k8s-credential']) {
                    sh '''
                        kubectl set image deployment/$DEPLOYMENT_NAME $DEPLOYMENT_NAME=$IMAGE_NAME -n $NAMESPACE
                        kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo '🔍 Verifying Kubernetes deployment...'
                withKubeConfig([credentialsId: 'k8s-credential']) {
                    sh '''
                        kubectl get pods -n $NAMESPACE
                        kubectl get svc -n $NAMESPACE
                    '''
                }
            }
        }
    }

    post {
        always {
            echo '🎉 Pipeline execution completed successfully.'
        }
    }
}

