pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = 'github-ssh'  // GitHub SSH credentials ID in Jenkins
        AWS_CREDENTIALS = 'aws-global-credentials'  // Updated AWS credentials ID
        AWS_REGION = 'ap-south-1' // Mumbai region
        ECR_REPO_NAME = 'webapp' // ECR repository name
        ECR_REGISTRY = '209479288689.dkr.ecr.ap-south-1.amazonaws.com'
        IMAGE_TAG = "${BUILD_ID}"
        FULL_IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}"
        K8S_MANIFEST_PATH = 'k8s'
        NAMESPACE = 'default'
        APP_NAME = 'webapp'
        EKS_CLUSTER_NAME = 'my-cluster' // Update if your EKS cluster name is different
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: env.GIT_CREDENTIALS, url: 'https://github.com/ranjeetchavan7/simple-webapp.git', branch: 'main'
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    echo "Setting up virtual environment and installing dependencies for ${env.APP_NAME}"
                    sh 'python3 -m venv venv'
                    sh '. venv/bin/activate && python3 -m pip install -r requirements.txt'
                    echo "Running tests for ${env.APP_NAME}"
                    if (fileExists("${env.APP_NAME}/tests")) {
                        sh ". venv/bin/activate && python3 -m unittest discover ${env.APP_NAME}/tests"
                    } else {
                        echo "No tests directory found. Skipping tests."
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${env.FULL_IMAGE_NAME}"
                    sh "docker build -f Dockerfile -t ${env.APP_NAME} ."
                    sh "docker tag ${env.APP_NAME} ${env.FULL_IMAGE_NAME}"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: env.AWS_CREDENTIALS]]) {
                    script {
                        echo "Logging into Amazon ECR"
                        sh "aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.ECR_REGISTRY}"

                        echo "Pushing Docker image to ECR: ${env.FULL_IMAGE_NAME}"
                        sh "docker push ${env.FULL_IMAGE_NAME}"
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: env.AWS_CREDENTIALS]]) {
                    script {
                        echo "Setting up kubeconfig for EKS"
                        sh "aws eks update-kubeconfig --region ${env.AWS_REGION} --name ${env.EKS_CLUSTER_NAME}"

                        echo "Deploying to EKS"
                        sh "kubectl apply -f ${env.K8S_MANIFEST_PATH}/deployment.yaml -n ${env.NAMESPACE}"
                        sh "kubectl apply -f ${env.K8S_MANIFEST_PATH}/service.yaml -n ${env.NAMESPACE}"
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    def serviceName = "${env.APP_NAME}-service"
                    def maxRetries = 5
                    def retryInterval = 30

                    for (int i = 0; i < maxRetries; i++) {
                        try {
                            def serviceInfo = sh(script: "kubectl get service ${serviceName} -n ${env.NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true).trim()
                            if (serviceInfo) {
                                echo "Service ${serviceName} is available at: ${serviceInfo}"
                                sh "curl --fail --show-error http://${serviceInfo}"
                                break
                            } else {
                                echo "LoadBalancer hostname for ${serviceName} not yet available. Retrying in ${retryInterval} seconds (${i + 1}/${maxRetries})..."
                                sleep time: retryInterval, unit: 'SECONDS'
                            }
                        } catch (Exception e) {
                            echo "Error checking service status: ${e.getMessage()}"
                            if (i < maxRetries - 1) {
                                sleep time: retryInterval, unit: 'SECONDS'
                            } else {
                                error "Failed to verify deployment of ${serviceName} after ${maxRetries} retries."
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed for ${env.APP_NAME}."
        }
        failure {
            echo "Pipeline failed for ${env.APP_NAME} :("
        }
        success {
            echo "Pipeline succeeded for ${env.APP_NAME}!"
        }
    }
}
