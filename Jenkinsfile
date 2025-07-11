pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = '8179958869'
        KUBECONFIG_CREDENTIALS_ID = '817995886'
        DOCKER_HUB_USERNAME = 'yamini8756'

        BACKEND_IMAGE = "${DOCKER_HUB_USERNAME}/ecommerce-backend"
        FRONTEND_IMAGE = "${DOCKER_HUB_USERNAME}/ecommerce-ui"

        BACKEND_REPO = 'https://github.com/yamini8756/Order-backend.git'
        FRONTEND_REPO = 'https://github.com/yamini8756/Order-frontend.git'
        BACKEND_BRANCH = 'main'
        FRONTEND_BRANCH = 'main'
    }

    stages {
        stage('Clone Backend') {
            steps {
                dir('backend') {
                    git url: "${BACKEND_REPO}", branch: "${BACKEND_BRANCH}"
                }
            }
        }

        stage('Clone Frontend') {
            steps {
                dir('frontend') {
                    git url: "${FRONTEND_REPO}", branch: "${FRONTEND_BRANCH}"
                }
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    bat 'mvnw.cmd clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    bat 'echo %PASSWORD% | docker login -u %USERNAME% --password-stdin'

                    bat "docker build -t %BACKEND_IMAGE% backend"
                    bat "docker push %BACKEND_IMAGE%"

                    bat "docker build -t %FRONTEND_IMAGE% frontend"
                    bat "docker push %FRONTEND_IMAGE%"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG_FILE')]) {
                    withEnv(["KUBECONFIG=$KUBECONFIG_FILE"]) {
                        bat 'kubectl apply -f k8s/mysql.yaml'
                        bat 'kubectl apply -f k8s/backend.yaml'
                        bat 'kubectl apply -f k8s/frontend.yaml'
                    }
                }
            }
        }
    }

    post {
        always {
            bat 'docker logout || exit 0'
        }
    }
}
