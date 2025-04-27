pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-south-1' // Change to your AWS region
        ECR_REPO = '209479288689.dkr.ecr.ap-south-1.amazonaws.com/my-repo' // Your ECR repository URL
        IMAGE_TAG = 'latest' // Tag for the image
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git repository'
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                echo 'Setting up virtual environment and installing dependencies for webapp'
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
                echo 'Running tests for webapp'
                script {
                    if (fileExists('tests')) {
                        sh 'pytest tests/'
                    } else {
                        echo 'No tests directory found. Skipping tests.'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${ECR_REPO}:${IMAGE_TAG}"
                sh 'docker build -f Dockerfile -t my-repo .'
                sh "docker tag my-repo ${ECR_REPO}:${IMAGE_TAG}"
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials', region: "${AWS_REGION}"]]) {
                    echo 'Logging into AWS ECR'
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
                    echo 'Pushing Docker image to ECR'
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials', region: "${AWS_REGION}"]]) {
                    echo 'Configuring kubectl for EKS'
                    sh 'aws eks update-kubeconfig --region ${AWS_REGION} --name your-cluster-name' // Update with your EKS cluster name
                    echo 'Deploying to EKS'
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
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
            // Any cleanup actions (e.g., removing Docker images, clearing caches)
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
