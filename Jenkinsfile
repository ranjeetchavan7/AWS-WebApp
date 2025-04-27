pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '209479288689.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPO_NAME = 'my-repo'
        EKS_CLUSTER_NAME = 'my-cluster'
        IMAGE_TAG = 'latest'
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
                sh '''
                docker build -f Dockerfile -t ${ECR_REPO_NAME} .
                docker tag ${ECR_REPO_NAME} ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    echo 'Logging into AWS ECR'
                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    echo 'Pushing Docker image to ECR'
                    docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    echo 'Configuring kubectl for EKS'
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export AWS_DEFAULT_REGION=${AWS_REGION}

                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region $AWS_DEFAULT_REGION

                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                    echo 'Applying Kubernetes deployment and service files'
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    echo 'Verifying deployment'
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export AWS_DEFAULT_REGION=${AWS_REGION}

                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region $AWS_DEFAULT_REGION

                    kubectl get pods
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up'
            sh 'docker rmi ${ECR_REPO_NAME} || true'
        }
    }
}
