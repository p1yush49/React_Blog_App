pipeline {
    agent any

    tools {
        nodejs 'nodejs-24'
    }

    environment {
        APP_NAME = 'react-blog-app'
        IMAGE_TAG = "${APP_NAME}:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/p1yush49/React_Blog_App.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }

        stage('Deploy to K3s') {
            steps {
                sh """
                    docker save ${IMAGE_TAG} -o /tmp/${APP_NAME}.tar
                    sudo k3s ctr images import /tmp/${APP_NAME}.tar
                    rm /tmp/${APP_NAME}.tar
                    kubectl set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE_TAG} -n react-apps || echo 'Deployment not found, will create via Terraform'
                """
            }
        }
    }

    post {
        success {
            echo "Build #${BUILD_NUMBER} deployed successfully!"
        }
        failure {
            echo "Build #${BUILD_NUMBER} failed."
        }
    }
}