pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPOSITORY = 'student-service'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    python3 -m py_compile app.py
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t student-service:latest .'
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
                    aws ecr get-login-password --region "$AWS_REGION" |
                    docker login --username AWS --password-stdin \
                    "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"
                '''
            }
        }

        stage('Docker Tag') {
            steps {
                sh '''
                    ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
                    ECR_URI="$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY"
                    docker tag student-service:latest "$ECR_URI:latest"
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
                    ECR_URI="$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY"
                    docker push "$ECR_URI:latest"
                '''
            }
        }
	stage('Deploy') {
            steps {
                sh '''
                    docker rm -f student-service-container || true
                    docker run -d \
                        --name student-service-container \
                        -p 5000:5000 \
                        student-service:latest
                '''
            }
        }
    }
}
