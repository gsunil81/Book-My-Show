pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
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
            credentialsId: 'git-creds',
            url: 'https://github.com/gsunil81/Book-My-Show.git'
      }
    }

    stage('Install Dependencies (NPM)') {
      steps {
        sh 'npm install'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        sh 'sonar-scanner'
      }
    }

    stage('Docker Build & Push to DockerHub') {
      steps {
        sh '''
          docker build -t yourdockerhubusername/bookmyshow-app .
          echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
          docker push yourdockerhubusername/bookmyshow-app
        '''
      }
    }

    stage('Deploy to Docker Container') {
      steps {
        sh 'docker run -d -p 3000:3000 yourdockerhubusername/bookmyshow-app'
      }
    }
  }

  post {
    failure {
      echo 'Pipeline failed.'
    }
  }
}
