pipeline {
  agent any
  tools {
    maven 'Maven 3.9.4'
  }
  triggers { pollSCM('H/5 * * * *') }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }
    stage('SonarQube analysis') {
      steps {
        script {
          // 'SonarScanner' debe coincidir EXACTO con el Name que diste en Global Tool Configuration → SonarQube Scanner
          // y type debe ser la clase que Jenkins listó como válida: hudson.plugins.sonar.SonarRunnerInstallation
          def scannerHome = tool name: 'SonarScanner', type: hudson.plugins.sonar.SonarRunnerInstallation
          withSonarQubeEnv('MySonarQube') {
            // Ejecuta el binario del scanner
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }
    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
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
