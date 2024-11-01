// Define services and their Docker image names
def services = [
    'ui-web-app-reactjs': 'ui',
    'wishlist-microservice-python': 'wishlist',
    'zuul-api-gateway': 'zuul',
    'cart-microservice-nodejs': 'cart',
    'offers-microservice-spring-boot': 'offer',
    'shoes-microservice-spring-boot': 'shoe'                     
]
def buildImages = true

pipeline {
    agent any

    parameters {
        booleanParam(name: 'buildImages', defaultValue: true, description: 'Build Docker images?')
    }
    
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
            when {
                expression { params.buildImages } // The stage will run if buildImages is true
            }
            steps {
                script {
                    // Loop through directories to build Docker images
                    sh "docker --version"
                    services.each { dir, image ->
                        echo "Processing directory: ${dir} with image name: ${image}"
                        if (fileExists("${dir}/Dockerfile")) {
                            echo "Dockerfile found in ${dir}, attempting to build ${DOCKER_REPO}/${image}:latest"
                            sh """
                                cd ${dir}
                                echo "Current working directory: \$(pwd)"
                                ls -la
                                docker build -t ${DOCKER_REPO}/${image}:latest .
                                echo "Successfully built ${DOCKER_REPO}/${image}:latest"
                            """
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
                        sh """
                            echo "Pushing Docker image: ${DOCKER_REPO}/${image}:latest"
                            docker push ${DOCKER_REPO}/${image}:latest
                        """      
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy each service using the deployment file in its respective directory
                    withKubeConfig(caCertificate: '', clusterName: 'minikube', contextName: '', credentialsId: 'K8-token', namespace: 'jenkins', restrictKubeConfigAccess: false, serverUrl: 'https://127.0.0.1:52737') {
                        services.each { dir, service ->
                            sh "kubectl apply -f ${dir}/kube.yaml --namespace=${KUBE_NAMESPACE}"
                        }
                    }
                }
            }
        }
    }
}
