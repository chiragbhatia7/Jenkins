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
                    // Remove existing network if it exists
                    sh 'docker network rm ${NETWORK_NAME} || true'
                    // Create new network
                    sh 'docker network create ${NETWORK_NAME}'
                    // Connect Jenkins container to the network
                    sh 'docker network connect ${NETWORK_NAME} jenkins || true'
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
                    // Stop and remove existing container
                    sh 'docker stop ${DOCKER_IMAGE} || true'
                    sh 'docker rm ${DOCKER_IMAGE} || true'
                    
                    // Verify network exists
                    sh 'docker network inspect ${NETWORK_NAME}'
                    
                    // Run new container
                    sh '''
                        docker run -d \
                            -p 8081:80 \
                            --name ${DOCKER_IMAGE} \
                            --network ${NETWORK_NAME} \
                            ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                    
                    // Verify container is running
                    sh 'docker ps | grep ${DOCKER_IMAGE}'
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    // Wait for container to be ready
                    sh 'sleep 10'
                    
                    // Check container status
                    sh '''
                        docker ps | grep ${DOCKER_IMAGE}
                        docker logs ${DOCKER_IMAGE}
                        curl -f http://localhost:8081 || exit 1
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Cleanup in case of any result
                sh '''
                    docker stop ${DOCKER_IMAGE} || true
                    docker rm ${DOCKER_IMAGE} || true
                    docker network disconnect ${NETWORK_NAME} jenkins || true
                    docker network rm ${NETWORK_NAME} || true
                '''
            }
        }
    }
}