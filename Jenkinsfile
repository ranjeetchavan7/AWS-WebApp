pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
        ECR_REPO_URI = '209479288689.dkr.ecr.ap-south-1.amazonaws.com/my-repo'
        CLUSTER_NAME = 'my-cluster'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git repository'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh 'docker build -f Dockerfile -t my-repo .'
                sh "docker tag my-repo ${ECR_REPO_URI}:latest"
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([aws(credentialsId: 'aws-credentials', region: 'ap-south-1')]) {
                    echo 'Logging into AWS ECR'
                    sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECR_REPO_URI}"
                    echo 'Pushing Docker image to ECR'
                    sh "docker push ${ECR_REPO_URI}:latest"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([aws(credentialsId: 'aws-credentials', region: 'ap-south-1')]) {
                    echo 'Configuring kubectl for EKS'
                    sh "aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${CLUSTER_NAME}"

                    // Apply Kubernetes deployment and service files from 'k8s' directory
                    echo 'Applying Kubernetes deployment and service files'
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment'
                sh 'kubectl get pods'
                sh 'kubectl get svc'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up'
            // Clean up Docker images (optional, can be adjusted)
            sh 'docker rmi my-repo || true'
        }
    }
}
