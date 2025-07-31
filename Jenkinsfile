pipeline {
  agent any

  environment {
    APP = "hrms-backend"
    IMAGE = "cloudansh/hrms-backend:latest"
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t ${IMAGE} .'
      }
    }

    stage('Trivy Scan') {
      steps {
        sh "trivy image --severity CRITICAL,HIGH ${IMAGE} || true"
      }
    }

    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-hub-creds',
          usernameVariable: 'DOCKER_USERNAME',
          passwordVariable: 'DOCKER_PASSWORD'
        )]) {
          sh '''
            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            docker push ${IMAGE}
          '''
        }
      }
    }

    stage('Docker Run on EC2') {
      steps {
        sh 'docker stop ${APP} || true && docker rm ${APP} || true'
        sh 'docker run -d --name ${APP} -p 5203:5203 ${IMAGE}'
      }
    }
  }

  post {
    failure {
      echo "❌ Pipeline failed. Check logs."
    }
  }
}
