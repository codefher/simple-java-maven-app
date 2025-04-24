pipeline {
  agent any
  tools {
    maven 'Maven 3.9.4'
    sonar  'SonarScanner'    // tu instalación de SonarQube Scanner
  }
  environment {
    IMAGE_NAME           = "codefher/simple-java-maven-app"
    REGISTRY_CREDENTIAL  = 'dockerhub-creds'
    SONARQUBE_SERVER     = 'MySonarQube'
    SONAR_PROJECT_KEY    = 'simple-java-maven-app'
    SONAR_PROJECT_NAME   = 'Simple Java Maven App'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Sonar Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
          sh "mvn sonar:sonar \
              -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
              -Dsonar.projectName='${SONAR_PROJECT_NAME}'"
        }
      }
    }
    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Build JAR') {
      steps { sh 'mvn clean package' }
    }
    stage('Build Docker Image') {
      steps {
        script { dockerImage = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}") }
      }
    }
    stage('Push to Docker Hub') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', REGISTRY_CREDENTIAL) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Archive Artifacts') {
      steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
    }
    stage('SonarQube analysis') {
      steps {
        withSonarQubeEnv('MySonarQube') {      // debe coincidir con el Name que pusiste en Configure System → SonarQube servers
          sh 'sonar-scanner'                   // arrancará el scanner instalado vía tools { sonar … }
        }
      }
    }
  }
  post {
    success {
      slackSend color: 'good',
        message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} passed Quality Gate"
    }
    failure {
      slackSend color: 'danger',
        message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} broken Quality Gate"
    }
  }
}
