pipeline {
    agent any

    environment {
        REGISTRY = "crimsonai" // Replace with your DockerHub username
        IMAGE_NAME = "node-app-argo" // Replace with your image name
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials') // Jenkins DockerHub credentials ID
        KUBE_CONTEXT = "minikube"
        KUBECONFIG = '/root/.kube/config'
        GIT_REPO = "https://github.com/jcrim1/kube-web-deploy.git" // Replace with your GitHub repo URL
    }

    stages {
      
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
                   sh 'kubectl apply -f dev/deployment.yaml'
                   sh 'kubectl apply -f dev/service.yaml'
                }
            }
        }

        stage('Sync with ArgoCD') {
            steps {
                script {
                    // Sync the deployment with ArgoCD
                    sh 'argocd login --username admin --password p7OgD85j4kOZ34KY localhost:8089 --insecure'
                    sh 'argocd app sync nodejs-app '
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
