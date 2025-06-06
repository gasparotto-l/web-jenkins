pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'gasparottoluo'  // Seu usuário Docker Hub
        APP_NAME = 'web-app'
        KUBE_CONFIG = credentials('rancher-kubeconfig')  // Usando sua credencial
        DOCKER_CREDS = credentials('docker-creds')  // Usando sua credencial Docker
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
                    sh "echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin"
                    
                    // Build da imagem
                    docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'docker-creds') {
                        docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}").push()
                        docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Configura o kubectl com o kubeconfig
                    withEnv(["KUBECONFIG=${KUBE_CONFIG}"]) {
                        // Atualiza a imagem no deployment.yaml
                        sh """
                            sed -i 's|image:.*|image: ${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}|g' deployment.yaml
                        """
                        
                        // Aplica o deployment
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl rollout status deployment/web-app --timeout=2m'
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
            slackSend(color: 'danger', message: "Falha na pipeline: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}