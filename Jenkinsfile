pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
        ECR_REPO = 'my-repo'
        IMAGE_TAG = '1'
        ECR_URI = "209479288689.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    echo 'Setting up virtual environment and installing dependencies for webapp'
                    sh 'python3 -m venv venv'
                    sh '. venv/bin/activate && pip install -r requirements.txt'
                    
                    echo 'Running tests for webapp'
                    // Add test commands if any, for now it will skip if no tests are found
                    if (fileExists('tests')) {
                        sh 'pytest tests'
                    } else {
                        echo 'No tests directory found. Skipping tests.'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${ECR_URI}"
                    sh 'docker build -f Dockerfile -t my-repo .'
                    sh "docker tag my-repo ${ECR_URI}"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    script {
                        // Login to ECR
                        sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin 209479288689.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                        
                        // Push image to ECR
                        sh "docker push ${ECR_URI}"
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    script {
                        // Set Kubernetes credentials using AWS EKS
                        sh 'aws eks --region ap-south-1 update-kubeconfig --name my-cluster'
                        
                        // Deploy Docker image to EKS
                        echo 'Deploying Docker image to EKS'
                        sh '''
                        kubectl apply -f kubernetes/deployment.yaml
                        kubectl set image deployment/my-deployment my-container=209479288689.dkr.ecr.ap-south-1.amazonaws.com/my-repo:1
                        '''
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo 'Verifying the deployment on EKS'
                    sh '''
                    kubectl rollout status deployment/my-deployment
                    kubectl get pods
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
