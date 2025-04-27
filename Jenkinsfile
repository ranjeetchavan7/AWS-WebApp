pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' // Set your AWS region here
    }

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout scm
            }
        }

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
                echo 'Building Docker image: 209479288689.dkr.ecr.ap-south-1.amazonaws.com/my-repo:latest'
                sh 'docker build -f Dockerfile -t my-repo .'
                sh 'docker tag my-repo 209479288689.dkr.ecr.ap-south-1.amazonaws.com/my-repo:latest'
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    echo 'Logging into AWS ECR'
                    sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin 209479288689.dkr.ecr.ap-south-1.amazonaws.com'
                    echo 'Pushing Docker image to ECR'
                    sh 'docker push 209479288689.dkr.ecr.ap-south-1.amazonaws.com/my-repo:latest'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    echo 'Configuring kubectl for EKS'
                    sh 'aws eks update-kubeconfig --region ${AWS_REGION} --name my-cluster'
                    echo 'Deploying application to EKS cluster'
                    // Add your kubectl deployment command here, e.g.
                    // sh 'kubectl apply -f deployment.yaml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment'
                // Add verification steps here, e.g., checking pod status
                // sh 'kubectl get pods'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up'
            // Add any cleanup steps here if needed
        }

        failure {
            echo 'Deployment failed!'
        }
    }
}
