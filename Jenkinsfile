pipeline {
    agent any

    environment {
        DOCKER_REPO = 'khaira23/flaskapp'
        DEPLOYMENT_NAME = 'flask-deployment'
        NAMESPACE = 'default'
    }

    stages {

        stage('Set Version Tag') {
            steps {
                script {
                    // Generate a version tag based on build number or timestamp
                    VERSION = "v${BUILD_NUMBER}"
                    IMAGE_NAME = "${DOCKER_REPO}:${VERSION}"
                    echo "🧾 New Image Version: ${IMAGE_NAME}"
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

        stage('Update Kubernetes Deployment') {
            steps {
                echo '☸️ Updating deployment in Kubernetes...'
                withKubeConfig([credentialsId: 'k8s-credential']) {
                    sh '''
                        # Update the deployment image with the new version
                        kubectl set image deployment/$DEPLOYMENT_NAME $DEPLOYMENT_NAME=$IMAGE_NAME -n $NAMESPACE
                        
                        # Wait for rollout to complete
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

