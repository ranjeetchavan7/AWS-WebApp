pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '209479288689'
        AWS_DEFAULT_REGION = 'ap-south-1'
        ECR_REPOSITORY = 'my-repo'  // Updated ECR repo name
        IMAGE_TAG = '1'
        GIT_REPO = 'https://github.com/ranjeetchavan7/AWS-WebApp.git'  // Updated Git repo URL
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-ssh', url: GIT_REPO, branch: 'main'
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    echo 'Setting up virtual environment and installing dependencies for webapp'
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        python3 -m pip install -r requirements.txt
                    '''
                    
                    echo 'Running tests for webapp'
                    if (fileExists('tests')) {
                        sh '''
                            . venv/bin/activate
                            pytest tests
                        '''
                    } else {
                        echo 'No tests directory found. Skipping tests.'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                    sh "docker build -f Dockerfile -t ${ECR_REPOSITORY} ."
                    sh "docker tag ${ECR_REPOSITORY} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    script {
                        echo 'Logging into Amazon ECR'
                        sh '''
                            aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
                        '''

                        echo "Pushing Docker image to ECR: ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                        sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    echo 'Deploying application to EKS cluster...'
                    // Add your kubectl apply or Helm deployment commands here
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo 'Verifying the deployment on EKS cluster...'
                    // Add your kubectl get pods/service verification commands here
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed for AWS-WebApp.'
        }
        failure {
            echo 'Pipeline failed for AWS-WebApp :('
        }
    }
}
