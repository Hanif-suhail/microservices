pipeline {
    agent any
    
    environment{
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
        DOCKER_REPO = 'hanifsuhail'
        KUBE_NAMESPACE = 'jenkins'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'feature', credentialsId: 'Git-Cred', url: 'https://github.com/Hanif-suhail/microservices.git'
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    // Define services and their Docker image names
                    def services = [
                        'ui-web-app-reactjs': 'ui-web-app',
                        'wishlist-microservice-python': 'wishlist-service',
                        'zuul-api-gateway': 'zuul-gateway',
                        'cart-microservice-nodejs': 'cart-service',
                        'offers-microservice-spring-boot': 'offers-service',
                        'shoes-microservice-spring-boot': 'shoes-service'
                    ]
                    // Loop through directories to build Docker images
                    services.each { dir, image ->
                        if (fileExists("${dir}/Dockerfile")) {
                            dir("${dir}") {
                                // Build the Docker image
                                docker.build("${DOCKER_REPO}/${image}:latest", ".")
                            }
                        }
                    }
                }
            }
        }
        
        stage('Push Docker Images') {
            steps {
                script {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    // Define services and their Docker image names again for pushing
                    def services = [
                        'ui-web-app-reactjs': 'ui-web-app',
                        'wishlist-microservice-python': 'wishlist-service',
                        'zuul-api-gateway': 'zuul-gateway',
                        'cart-microservice-nodejs': 'cart-service',
                        'offers-microservice-spring-boot': 'offers-service',
                        'shoes-microservice-spring-boot': 'shoes-service'
                    ]
                    // Loop through directories to push Docker images to the registry
                    services.each { dir, image ->
                        if (fileExists("${dir}/Dockerfile")) {
                            dir("${dir}") {
                                // Push the Docker image
                                docker.image("${DOCKER_REPO}/${image}:latest").push()
                            }
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def services = [
                        'ui-web-app-reactjs': 'ui-web-app',
                        'wishlist-microservice-python': 'wishlist-service',
                        'zuul-api-gateway': 'zuul-gateway',
                        'cart-microservice-nodejs': 'cart-service',
                        'offers-microservice-spring-boot': 'offers-service',
                        'shoes-microservice-spring-boot': 'shoes-service'
                    ]
                    // Deploy each service using the deployment file in its respective directory
                    withKubeConfig(caCertificate: '', clusterName: 'minikube', contextName: '', credentialsId: 'K8-token', namespace: 'jenkins', restrictKubeConfigAccess: false, serverUrl: 'https://127.0.0.1:45574') {
                        services.each { dir, service ->
                            sh "kubectl apply -f ${dir}/kube.yaml --namespace=${KUBE_NAMESPACE}"
                        }
                    }
                }
            }
        }
    }
}
