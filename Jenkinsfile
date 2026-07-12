pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        AWS_REGION = 'ap-south-2'
        ECR_REPO = '239762172856.dkr.ecr.ap-south-2.amazonaws.com/java-app'
        IMAGE_TAG = 'v1'
        CLUSTER_NAME = 'assignment-cluster'
        APP_NAME = 'java-project'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Java Application') {
            steps {
                sh '''
                mvn clean package -DskipTests
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $APP_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password \
                --region $AWS_REGION | \
                docker login \
                --username AWS \
                --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag \
                $APP_NAME:$IMAGE_TAG \
                $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push \
                $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Configure EKS Access') {
            steps {
                sh '''
                aws eks update-kubeconfig \
                --region $AWS_REGION \
                --name $CLUSTER_NAME
                '''
            }
        }

        stage('Deploy Application using Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods
                kubectl get svc
                '''
            }
        }
    }
}
