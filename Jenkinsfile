pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '093366266423.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPOSITORY = 'zomato'
        ECR_IMAGE = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}"
        EKS_CLUSTER = 'zomato-eks'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/BHARTI-BHUTE/Zomato.git'
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    node --version
                    npm --version
                    npm install
                    npm run build
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${ECR_IMAGE} .
                    docker images
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push ${ECR_IMAGE}
                '''
            }
        }

        stage('Deploy Staging') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                        --region ${AWS_REGION} \
                        --name ${EKS_CLUSTER}

                    kubectl apply -f Kubernetes/deployment.yaml \
                        -n staging

                    kubectl apply -f Kubernetes/service.yaml \
                        -n staging

                    kubectl set image deployment/zomato \
                        zomato=${ECR_IMAGE} \
                        -n staging

                    kubectl rollout status deployment/zomato \
                        -n staging \
                        --timeout=300s
                '''
            }
        }

        stage('Validate Staging') {
            steps {
                sh '''
                    echo "===== STAGING PODS ====="
                    kubectl get pods -n staging

                    echo "===== STAGING DEPLOYMENT ====="
                    kubectl get deployment -n staging

                    echo "===== STAGING SERVICE ====="
                    kubectl get svc -n staging

                    kubectl rollout status deployment/zomato \
                        -n staging \
                        --timeout=300s
                '''
            }
        }

        stage('Deploy Production') {
            steps {
                sh '''
                    kubectl apply -f Kubernetes/deployment.yaml \
                        -n production

                    kubectl apply -f Kubernetes/service.yaml \
                        -n production

                    kubectl set image deployment/zomato \
                        zomato=${ECR_IMAGE} \
                        -n production

                    kubectl rollout status deployment/zomato \
                        -n production \
                        --timeout=300s
                '''
            }
        }

        stage('Production Validation') {
            steps {
                sh '''
                    echo "===== PRODUCTION PODS ====="
                    kubectl get pods -n production

                    echo "===== PRODUCTION DEPLOYMENT ====="
                    kubectl get deployment -n production

                    echo "===== PRODUCTION SERVICE ====="
                    kubectl get svc -n production

                    kubectl rollout status deployment/zomato \
                        -n production \
                        --timeout=300s
                '''
            }
        }
    }

    post {
        success {
            echo "Zomato CI/CD Pipeline completed successfully."
            echo "Docker Image: ${ECR_IMAGE}"
        }

        failure {
            echo "Zomato CI/CD Pipeline failed."
        }
    }
}
