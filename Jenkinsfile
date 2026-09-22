pipeline {

    agent {
        label 'docker-agent'
    }

    environment {
        DOCKER_IMAGE = 'abhishekgavane/jenkins-docker-cicd'
        DOCKER_CREDENTIALS = 'dockerhub-creds'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    python3 --version
                    pip3 install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    pytest -v
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Docker Test') {
            steps {
                echo 'Testing Docker image...'
                sh '''
                    docker run -d --name jenkins-test -p 5000:5000 ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    sleep 5

                    curl -f http://localhost:5000/health

                    docker stop jenkins-test
                    docker rm jenkins-test
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                    docker image prune -f
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD pipeline failed. Check the stage logs.'
        }
    }
}
