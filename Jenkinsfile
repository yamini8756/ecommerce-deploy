pipeline {
    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-17'  // base image with Maven + Java 17
            args '-v /var/run/docker.sock:/var/run/docker.sock'  // allow Docker inside Docker
        }
    }

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
        stage('Install Node for Frontend') {
            steps {
                sh 'apt-get update && apt-get install -y curl'
                sh 'curl -fsSL https://deb.nodesource.com/setup_18.x | bash -'
                sh 'apt-get install -y nodejs'
            }
        }

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
                    sh './mvnw clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'

                    sh 'docker build -t $BACKEND_IMAGE backend'
                    sh 'docker push $BACKEND_IMAGE'

                    sh 'docker build -t $FRONTEND_IMAGE frontend'
                    sh 'docker push $FRONTEND_IMAGE'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG_FILE')]) {
                    withEnv(["KUBECONFIG=$KUBECONFIG_FILE"]) {
                        sh 'kubectl apply -f k8s/mysql.yaml'
                        sh 'kubectl apply -f k8s/backend.yaml'
                        sh 'kubectl apply -f k8s/frontend.yaml'
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
