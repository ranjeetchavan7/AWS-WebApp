pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '209479288689.dkr.ecr.ap-south-1.amazonaws.com/my-repo'
        IMAGE_TAG = '1'
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
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                '''
                echo 'Running tests for webapp'
                script {
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
                echo "Building Docker image: ${ECR_REPO}:${IMAGE_TAG}"
                sh '''
                    docker build -f Dockerfile -t my-repo .
                    docker tag my-repo ${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([aws(credentialsId: 'aws-credentials', region: AWS_REGION)]) {
                    echo 'Logging into ECR'
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([aws(credentialsId: 'aws-credentials', region: AWS_REGION)]) {
                    echo 'Updating kubeconfig to access EKS cluster'
                    sh '''
                        aws eks --region ${AWS_REGION} update-kubeconfig --name my-cluster
                    '''
                    echo 'Deploying Docker image to EKS'
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying the deployment on EKS'
                sh '''
                    kubectl get pods
                    kubectl get svc
                '''
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
    }
}
