pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REPO = "<account-id>.dkr.ecr.ap-south-1.amazonaws.com/test-app"
    CLUSTER_NAME = "test-eks"
  }

  stages {

    stage('Checkout') {
      steps {
        git '<your-github-repo>'
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t test-app .'
      }
    }

    stage('Push to ECR') {
      steps {
        sh '''
        aws ecr get-login-password --region $AWS_REGION |
        docker login --username AWS --password-stdin $ECR_REPO
        docker tag test-app:latest $ECR_REPO:latest
        docker push $ECR_REPO:latest
        '''
      }
    }

    stage('Deploy to EKS') {
      steps {
        sh '''
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
        kubectl apply -f k8s/
        '''
      }
    }
  }
}

