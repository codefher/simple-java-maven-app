pipeline {
  agent {
    docker {
      image 'maven:3.8.8-openjdk-17'
      args  '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Test') {
      steps {
        // dentro del contenedor ya hay Java y mvn
        sh 'chmod +x mvnw'
        sh './mvnw clean package'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("codefher/spring-web-service:${env.BUILD_NUMBER}")
          docker.withRegistry('', 'dockerhub-creds') {
            dockerImage.push()
          }
        }
      }
    }

    // … deploy como antes …
  }

  post {
    always { archiveArtifacts artifacts: 'service.log', allowEmptyArchive: true }
  }
}
