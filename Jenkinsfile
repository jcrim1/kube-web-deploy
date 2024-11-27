pipeline {
    agent any

    environment {
        REGISTRY = "crimsonai" // Replace with your DockerHub username
        IMAGE_NAME = "node-app-argo" // Replace with your image name
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials') // Jenkins DockerHub credentials ID
        KUBE_CONTEXT = "minikube"
        GIT_REPO = "https://github.com/jcrim1/kube-web-deploy.git" // Replace with your GitHub repo URL
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the code from GitHub
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    // Login to DockerHub using Jenkins credentials
                    withDockerRegistry(credentialsId: 'dockerhub-credentials', url: '') {
                        sh 'docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}'
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    // Set Kube Context to Minikube
                    sh 'kubectl config use-context ${KUBE_CONTEXT}'

                    // Apply Kubernetes YAML files for deployment
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                    sh 'kubectl apply -f k8s/ingress.yaml'
                }
            }
        }

        stage('Sync with ArgoCD') {
            steps {
                script {
                    // Sync the deployment with ArgoCD
                    sh 'argocd app sync nodejs-app --auth-token ${ARGOCD_TOKEN} --grpc-web'
                }
            }
        }

        stage('Clean Up') {
            steps {
                // Clean up old Docker images locally (optional)
                sh 'docker image prune -f'
            }
        }
    }

    post {
        always {
            echo 'Pipeline Finished.'
        }
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
