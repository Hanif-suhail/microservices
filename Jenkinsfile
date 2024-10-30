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
                    sh "docker --version"
                    try {
                        dir('ui-web-app-reactjs') {
                            docker.build("test-image:latest", ".")
                        }
                        echo "Docker build test passed."
                    } catch (Exception e) {
                        echo "Docker build test failed: ${e.getMessage()}"
                        error("Docker is not accessible to Jenkins.")
                    }
                    services.each { dir, image ->
                        echo "Processing directory: ${dir} with image name: ${image}"
                        if (fileExists("${dir}/Dockerfile")) {
                            echo "Dockerfile found in ${dir}, attempting to build ${DOCKER_REPO}/${image}:latest"
                            dir("${dir}") {
                                try {
                                    docker.build("${DOCKER_REPO}/${image}:latest", ".")
                                    echo "Successfully built ${DOCKER_REPO}/${image}:latest"
                                } catch (Exception e) {
                                    echo "Failed to build Docker image for ${dir}: ${e.getMessage()}"
                                }
                            }
                        } else {
                            echo "Dockerfile not found in ${dir}, skipping build"
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
