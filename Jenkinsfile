pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'gasparottoluo'
        APP_NAME = 'web-jenkins'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CONFIG = credentials('rancher-kubeconfig')  
        DOCKER_CREDS = credentials('docker-creds')       
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin"
                    
                    bat "docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} -t ${DOCKER_REGISTRY}/${APP_NAME}:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    bat "docker push ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
                    bat "docker push ${DOCKER_REGISTRY}/${APP_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    bat """
                        powershell -Command "(Get-Content deployment.yaml) -replace 'image: .*', 'image: ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}' | Set-Content deployment.yaml"
                    """

                    withCredentials([file(credentialsId: 'rancher-kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                        bat "set KUBECONFIG=%KUBECONFIG_FILE% && kubectl apply -f deployment.yaml"
                        bat "set KUBECONFIG=%KUBECONFIG_FILE% && kubectl rollout status deployment/web-jenkins --timeout=2m"
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Aplicação implantada com sucesso!'
            echo "📌 Acesse: http://<seu-node-ip>:30007"
        }
        failure {
            echo '❌ Falha na pipeline!'
        }
    }
}
