pipeline {
    agent any

    environment {
        PROD_SERVER = '10.10.30.103'
        APP_PORT = '3000'
        IMAGE_NAME = 'juiceshop'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Pulling latest code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy to Production') {
            steps {
                echo 'Deploying to prod-vm...'
                sh '''
                    ssh -o StrictHostKeyChecking=no corplab@${PROD_SERVER} "
                        docker stop juiceshop || true
                        docker rm juiceshop || true
                        docker pull bkimminich/juice-shop
                        docker run -d --name juiceshop --restart always -p ${APP_PORT}:${APP_PORT} bkimminich/juice-shop
                    "
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying Juice Shop is running...'
                sh 'curl -s -o /dev/null -w "%{http_code}" http://${PROD_SERVER}:${APP_PORT} | grep -q 200 && echo "Deployment successful" || echo "Deployment failed"'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}