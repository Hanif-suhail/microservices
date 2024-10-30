// Define services and their Docker image names
def services = [
    'ui-web-app-reactjs': 'ui',
    'wishlist-microservice-python': 'wishlist',
    'zuul-api-gateway': 'zuul',
    'cart-microservice-nodejs': 'cart',
    'offers-microservice-spring-boot': 'offer',
    'shoes-microservice-spring-boot': 'shoe'                     
]

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
                git branch: 'main', credentialsId: 'Git-Cred', url: 'https://github.com/Hanif-suhail/microservices.git'
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    // Loop through directories to build Docker images
                    services.each { dir, image ->
                        echo "Processing directory: ${dir} with image name: ${image}"
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
