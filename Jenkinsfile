pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'gasparottoluo'
        APP_NAME = 'web-jenkins'
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
                    // Login no Docker Hub (Windows)
                    bat "echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin"
                    
                    // Build da imagem (Windows)
                    bat "docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    // Push da imagem (Windows)
                    bat "docker push ${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}"
                    bat "docker push ${DOCKER_REGISTRY}/${APP_NAME}:latest"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Substituição da imagem (Windows)
                    bat """
                        powershell -Command "(Get-Content deployment.yaml) -replace 'image:.*', 'image: ${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}' | Set-Content deployment.yaml"
                    """
                    
                    // Aplica o deployment (Windows)
                    bat "kubectl apply -f deployment.yaml"
                    bat "kubectl rollout status deployment/web-app --timeout=2m"
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
            // Removido o slackSend que não está disponível
        }
    }
}