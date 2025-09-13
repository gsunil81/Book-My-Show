pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
    IMAGE_NAME = "sunil/bookmyshow"
  }

  stages {
    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }

    stage('Checkout Code') {
      steps {
        git credentialsId: 'github-creds',
            url: 'https://github.com/sunil/bookmyshow.git',
            branch: 'main'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        script {
          def scannerHome = tool 'SonarScanner'
          withSonarQubeEnv('sonarqube') {
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }

    stage('Install NPM Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Docker Cleanup') {
      steps {
        sh 'docker rm -f bookmyshow || true'
      }
    }

    stage('Docker Build & Push') {
      steps {
        script {
          docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
            def app = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
            app.push()
          }
        }
      }
    }

    stage('Deploy Container') {
      steps {
        sh "docker run -d -p 8080:8080 --name bookmyshow ${IMAGE_NAME}:${env.BUILD_NUMBER}"
      }
    }

    stage('Email Notification') {
      steps {
        mail to: 'your.email@example.com',
             subject: "Jenkins Build ${currentBuild.fullDisplayName}",
             body: "Build ${currentBuild.currentResult}: ${env.BUILD_URL}"
      }
    }
  }

  post {
    failure {
      mail to: 'your.email@example.com',
           subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: "Check console output: ${env.BUILD_URL}"
    }
  }
}
