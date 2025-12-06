pipeline {
    agent any

    environment {
        DOCKERHUB = credentials('docker-creds')
        BACKEND_IMAGE = "kubemayurr/backend"
        FRONTEND_IMAGE = "kubemayurr/frontend"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Pulling repository from GitHub..."
                checkout scm
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    echo "Building Backend Image..."
                    sh """
                        docker build -t ${BACKEND_IMAGE}:latest ./backend
                    """
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    echo "Building Frontend Image..."
                    sh """
                        docker build -t ${FRONTEND_IMAGE}:latest ./frontend
                    """
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    echo "Logging into DockerHub..."
                    sh '''
                        echo "${DOCKERHUB_PSW}" | docker login -u "${DOCKERHUB_USR}" --password-stdin
                    '''
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                script {
                    echo "Pushing images to DockerHub..."
                    sh """
                        docker push ${BACKEND_IMAGE}:latest
                        docker push ${FRONTEND_IMAGE}:latest
                    """
                }
            }
        }

        stage('Update Compose Images (Optional for Prod)') {
            steps {
                script {
                    echo "Updating docker-compose.yml with latest images..."
                    sh """
                        sed -i 's|build: ./backend|image: ${BACKEND_IMAGE}:latest|' docker-compose.yml
                        sed -i 's|build: ./frontend|image: ${FRONTEND_IMAGE}:latest|' docker-compose.yml
                    """
                }
            }
        }

        stage('Deploy Using Docker Compose') {
            steps {
                script {
                    echo "Deploying application using docker-compose..."
                    sh """
                        docker-compose pull
                        docker-compose down
                        docker-compose up -d
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
    }
}

