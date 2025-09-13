pipeline {
    agent any

    environment {
        IMAGE_NAME = "bookmyshow-app"
        REGISTRY = "docker.io/yourusername" // Replace with your Docker Hub username
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t $REGISTRY/$IMAGE_NAME:latest ."
            }
        }

        stage('Push') {
            steps {
                echo "Pushing image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push $REGISTRY/$IMAGE_NAME:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying container..."
                sh "docker run -d -p 8080:8080 $REGISTRY/$IMAGE_NAME:latest"
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}