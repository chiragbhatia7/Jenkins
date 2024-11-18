pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'nginx-app'
        DOCKER_TAG = 'latest'
        NETWORK_NAME = 'jenkins-net'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Network') {
            steps {
                script {
                    sh '''
                        if ! docker network ls | grep -q ${NETWORK_NAME}; then
                            docker network create ${NETWORK_NAME}
                        fi
                    '''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dir('nginx') {
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    }
                }
            }
        }
        
        stage('Deploy Container') {
            steps {
                script {
                    sh '''
                        docker stop ${DOCKER_IMAGE} || true
                        docker rm ${DOCKER_IMAGE} || true
                        docker run -d -p 8081:80 --name ${DOCKER_IMAGE} --network ${NETWORK_NAME} ${DOCKER_IMAGE}:${DOCKER_TAG}
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
        cleanup {
            script {
                sh '''
                    # Keep the network if the build succeeds, remove it if it fails
                    if [ "${currentBuild.currentResult}" != "SUCCESS" ]; then
                        docker network rm ${NETWORK_NAME} || true
                    fi
                '''
            }
        }
    }
}