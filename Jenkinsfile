# Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'nginx-app'
        DOCKER_TAG = 'latest'
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
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} -f nginx/Dockerfile ."
                }
            }
        }
        
        stage('Deploy Container') {
            steps {
                script {
                    sh '''
                        docker stop ${DOCKER_IMAGE} || true
                        docker rm ${DOCKER_IMAGE} || true
                        docker run -d -p 8081:80 --name ${DOCKER_IMAGE} --network jenkins-net ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    sh '''
                        sleep 10
                        curl -f http://localhost:8081 || exit 1
                    '''
                }
            }
        }
    }
    
    post {
        failure {
            script {
                sh '''
                    docker stop ${DOCKER_IMAGE} || true
                    docker rm ${DOCKER_IMAGE} || true
                '''
            }
        }
    }
}