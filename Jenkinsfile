


pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'tewelljd-dockerhub'
        DOCKER_IMAGE = 'tewelljd/225-lab3-2'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        KUBECONFIG = credentials('tewelljd-fl26')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'tewelljd-dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    sed "s|tewelljd/225-lab3-2:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|g" \
                        deployment-dev.yaml > deployment-build.yaml

                    kubectl apply -f deployment-build.yaml
                '''
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                sh '''
                    kubectl rollout status deployment/dev-deployment \
                        --timeout=120s

                    kubectl get services
                    kubectl get ingress
                '''
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#builds',
                color: 'good',
                message: "SUCCESS: ${JOB_NAME} build #${BUILD_NUMBER}"
            )
        }

        failure {
            slackSend(
                channel: '#builds',
                color: 'danger',
                message: "FAILED: ${JOB_NAME} build #${BUILD_NUMBER}. Check Jenkins Console Output."
            )
        }

        always {
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
    }
}
