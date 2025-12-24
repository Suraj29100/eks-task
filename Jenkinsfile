pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_ACCOUNT = "427793686617"
        ECR_REPO = "test-nginx"
        CLUSTER_NAME = "test-eks"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/Suraj29100/eks-task.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${ECR_REPO}:${IMAGE_TAG} .'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                      aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                      aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                      aws configure set default.region ${AWS_REGION}

                      aws ecr get-login-password --region ${AWS_REGION} \
                      | docker login --username AWS --password-stdin ${ECR_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag & Push Image to ECR') {
            steps {
                sh '''
                  docker tag ${ECR_REPO}:${IMAGE_TAG} \
                  ${ECR_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}

                  docker push ${ECR_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                      aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                      aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                      aws configure set default.region ${AWS_REGION}

                      aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}

                      kubectl apply -f k8s/
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully"
        }
        failure {
            echo "CI/CD Pipeline failed"
        }
    }
}

