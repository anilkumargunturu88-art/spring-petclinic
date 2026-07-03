pipeline {
    agent any

    environment {
        AWS_REGION = "ap-southeast-1"
        ACCOUNT_ID = "575005867523"
        ECR_REPO = "appecr"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/anilkumargunturu88-art/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t spring-petclinic:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                export AWS_PAGER=""
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker tag spring-petclinic:${IMAGE_TAG} \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}

                docker push \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['ec2-ssh']) {

                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@47.129.246.137 << EOF

                    export AWS_PAGER=""

                    aws ecr get-login-password --region ap-southeast-1 | docker login \
                    --username AWS \
                    --password-stdin \
                    575005867523.dkr.ecr.ap-southeast-1.amazonaws.com

                    docker pull \
                    575005867523.dkr.ecr.ap-southeast-1.amazonaws.com/appecr:latest

                    docker stop spring-petclinic || true
                    docker rm spring-petclinic || true

                    docker run -d \
                    --name spring-petclinic \
                    -p 8081:8080 \
                    575005867523.dkr.ecr.ap-southeast-1.amazonaws.com/appecr:latest

                    EOF
                    '''
                }
            }
        }

    }
}
