pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.15/bin:$PATH"
        AWS_REGION = "us-east-1"
        IMAGE_NAME = "449901517774.dkr.ecr.us-east-1.amazonaws.com/ttrend"
        VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage("Build") {
            steps {
                echo "----------- Build Started ----------"
                sh 'mvn clean package -DskipTests'
                echo "----------- Build Completed ----------"
            }
        }

        stage("Test") {
            steps {
                echo "----------- Test Started ----------"
                sh 'mvn test'
                echo "----------- Test Completed ----------"
            }
        }

        stage("Docker Build") {
            steps {
                echo '<--------------- Docker Build Started --------------->'
                sh 'docker build -t $IMAGE_NAME:$VERSION .'
                echo '<--------------- Docker Build Completed --------------->'
            }
        }

        stage("Login to ECR") {
            steps {
                echo '<--------------- ECR Login --------------->'
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin 449901517774.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage("Push to ECR") {
            steps {
                echo '<--------------- Push to ECR --------------->'
                sh 'docker push $IMAGE_NAME:$VERSION'
            }
        }

        stage("Deploy on EC2") {
            steps {
                echo '<--------------- Deployment Started --------------->'
                sh '''
                docker pull $IMAGE_NAME:$VERSION
                docker rm -f ttrend || true
                docker run -d --name ttrend -p 8000:8000 --restart unless-stopped $IMAGE_NAME:$VERSION
                '''
                echo '<--------------- Deployment Completed --------------->'
            }
        }
    }

    post {
        always {
            echo "Cleaning up unused Docker images..."
            sh 'docker image prune -f'
        }
    }
}