pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // Jenkins credential ID
        IMAGE_NAME = "sunil8179/bookmyshow"                    // DockerHub repo
        SONARQUBE_SERVER = "SonarQube"                         // Jenkins SonarQube server name
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

        stage('SonarQube Analysis (Quality Gate)') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Install Dependencies (NPM)') {
            steps {
                sh 'npm install'
            }
        }

        stage('Trivy FS Scan (Optional)') {
            when {
                expression { fileExists('package.json') }
            }
            steps {
                sh 'trivy fs . || true'
            }
        }

        stage('OWASP Dependency Check (Optional)') {
            steps {
                sh '''
                mkdir -p dependency-check
                wget https://github.com/jeremylong/DependencyCheck/releases/download/v8.4.0/dependency-check-8.4.0-release.zip
                unzip dependency-check-8.4.0-release.zip -d dependency-check
                dependency-check/dependency-check/bin/dependency-check.sh --project BookMyShow --scan .
                '''
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

        stage('Deploy to Kubernetes (EKS)') {
            when {
                expression { fileExists('k8s/deployment.yaml') }
            }
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl rollout status deployment/bookmyshow
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
