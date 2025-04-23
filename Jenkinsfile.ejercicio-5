pipeline {
  agent any
  tools { maven 'Maven 3.9.4' }
  triggers { pollSCM('H/5 * * * *') }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build') {
      steps { sh 'mvn clean package' }
    }
    stage('Archive') {
      steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
    }
  }
  post {
    success {
      slackSend color: 'good', message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} OK (<${env.BUILD_URL}|Ver>)"
    }
    failure {
      slackSend color: 'danger', message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} FALLÓ (<${env.BUILD_URL}|Ver>)"
    }
  }
}
