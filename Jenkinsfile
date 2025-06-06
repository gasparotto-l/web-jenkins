pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'gasparottoluo'
        APP_NAME = 'web-jenkins'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CONFIG = credentials('rancher-kubeconfig')  // Certifique-se que está mapeado corretamente
        DOCKER_CREDS = credentials('docker-creds')       // ID das credenciais no Jenkins
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
                    // Login no Docker Hub
                    bat "echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin"
                    
                    // Build da imagem
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
                    // Atualiza o arquivo deployment.yaml com a nova imagem
                    bat """
                        powershell -Command "(Get-Content deployment.yaml) -replace 'image: .*', 'image: ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}' | Set-Content deployment.yaml"
                    """

                    // Usa o kubeconfig para aplicar (caso precise setar KUBECONFIG)
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
