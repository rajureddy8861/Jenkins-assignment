```groovy
pipeline {

    agent any

    environment {
        AWS_REGION = 'us-east-1'
        EKS_CLUSTER = 'jenkins-eks'
        ECR_REPOSITORY = 'jenkinsandjava'
        AWS_ACCOUNT_ID = '518692945862'

        IMAGE_TAG = "${BUILD_NUMBER}"
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Unit Test') {
            steps {
                echo 'Running unit tests'
                sh 'mvn clean test'
            }
        }

        stage('Build') {
            steps {
                echo 'Building Java application'
                sh 'mvn package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('ECR Login') {
            steps {
                echo 'Logging into Amazon ECR'

                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                echo "Pushing image to ECR: ${IMAGE_NAME}:${IMAGE_TAG}"

                sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo "Deploying ${IMAGE_NAME}:${IMAGE_TAG} to EKS"

                sh '''
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${EKS_CLUSTER}

                    kubectl set image deployment/jenkinsandjava \
                      jenkinsandjava=${IMAGE_NAME}:${IMAGE_TAG}

                    kubectl rollout status deployment/jenkinsandjava \
                      --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying Kubernetes deployment'

                sh '''
                    kubectl get deployment jenkinsandjava
                    kubectl get pods -l app=jenkinsandjava -o wide
                    kubectl get svc jenkinsandjava-service
                '''
            }
        }
    }

    post {

        success {
            echo "CI/CD pipeline completed successfully"
            echo "Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
        }

        failure {
            echo "CI/CD pipeline failed"
        }
    }
}
```
