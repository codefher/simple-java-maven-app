pipeline {
  agent any
  tools { maven 'Maven 3.9.4' }
  environment {
    IMAGE_NAME = "codefher/simple-java-maven-app"
    REGISTRY_CREDENTIAL = 'dockerhub-creds'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build JAR') {
      steps { sh 'mvn clean package' }
    }
    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
        }
      }
    }
    stage('Push to Docker Hub') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', "${REGISTRY_CREDENTIAL}") {
            dockerImage.push()
          }
        }
      }
    }
    stage('Archive Artifacts') {
      steps {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }
  }
  post {
    success {
      slackSend color: 'good', message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} Image pushed (${IMAGE_NAME}:${env.BUILD_NUMBER})"
    }
    failure {
      slackSend color: 'danger', message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED"
    }
  }
}
