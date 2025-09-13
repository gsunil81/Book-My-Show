pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "sunil8179/bookmyshow"
        SONARQUBE_SERVER = "SonarQube"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code from GitHub') {
            steps {
                git branch: 'feature/devops-pipeline',
                    url: 'https://github.com/gsunil81/Book-My-Show.git'
            }
        }

        stage('Install Dependencies (NPM)') {
            steps {
                sh 'npm install'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . || true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Docker Build & Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
                        def app = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                        app.push()
                    }
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                sh '''
                docker rm -f bookmyshow || true
                docker run -d --name bookmyshow -p 3000:3000 sunil8179/bookmyshow:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        success {
            mail to: 'galisunilkumar81@gmail.com',
                 subject: "✅ Jenkins Build #${BUILD_NUMBER} Success",
                 body: "The BookMyShow pipeline completed successfully.\nCheck Jenkins for full logs."
        }
        failure {
            mail to: 'galisunilkumar81@gmail.com',
                 subject: "❌ Jenkins Build #${BUILD_NUMBER} Failed",
                 body: "The pipeline failed. Please check Jenkins logs and fix the errors."
        }
    }
}
