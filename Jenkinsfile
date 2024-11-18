pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'nginx-app'
        DOCKER_TAG = 'latest'
        NETWORK_NAME = 'jenkins-net'
        // Fixed HOST_IP retrieval for WSL2
        HOST_IP = sh(script: "hostname -I | awk '{print \$1}'", returnStdout: true).trim()
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
                    
                    // Run new container on the custom network
                    sh '''
                        docker run -d \
                            --network ${NETWORK_NAME} \
                            --name ${DOCKER_IMAGE} \
                            ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                    
                    // Verify container is running
                    sh 'docker ps | grep ${DOCKER_IMAGE}'
                    sh 'docker logs ${DOCKER_IMAGE}'
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    // Wait for container to be ready
                    sh 'sleep 15'
                    
                    // Perform health check using different methods
                    sh '''
                        # Try using HOST_IP
                        curl -f http://${HOST_IP}:80/health || \
                        # Try container IP from the network
                        CONTAINER_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${DOCKER_IMAGE}) && \
                        curl -f http://${CONTAINER_IP}:80/health || \
                        exit 1
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Cleanup
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
