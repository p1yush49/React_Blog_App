pipeline {
    agent any

    tools {
        nodejs 'nodejs-24'
    }

    environment {
        APP_NAME = 'react-blog-app'
        IMAGE_TAG = "${APP_NAME}:${BUILD_NUMBER}"
        KC = '--kubeconfig /etc/rancher/k3s/k3s.yaml'
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
                sh 'CI=false npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }

        stage('Import to K3s') {
            steps {
                sh """
                    docker save ${IMAGE_TAG} | sudo k3s ctr images import -
                """
            }
        }

        stage('Deploy to K3s') {
            steps {
                sh """
                    sudo kubectl ${KC} create namespace react-apps --dry-run=client -o yaml | sudo kubectl ${KC} apply -f -
                    sudo kubectl ${KC} get deployment ${APP_NAME} -n react-apps 2>/dev/null && \
                    sudo kubectl ${KC} set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE_TAG} -n react-apps || \
                    sudo kubectl ${KC} create deployment ${APP_NAME} --image=${IMAGE_TAG} -n react-apps
                    sudo kubectl ${KC} expose deployment ${APP_NAME} --port=80 --target-port=80 -n react-apps --dry-run=client -o yaml | sudo kubectl ${KC} apply -f -
                """
            }
        }

        stage('Cleanup') {
            steps {
                sh """
                    docker image prune -f
                    docker images ${APP_NAME} --format '{{.Tag}}' | sort -rn | tail -n +4 | xargs -r -I {} docker rmi ${APP_NAME}:{} || true
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
