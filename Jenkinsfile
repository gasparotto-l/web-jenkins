pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'web-app'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}:latest")
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl rollout status deployment/web-app'
                }
            }
        }
    }
    post {
        success {
            echo '✅ Aplicação implantada! Acesse: http://localhost:30007'
        }
    }
}