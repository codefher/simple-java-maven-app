pipeline {
  agent {
    docker {
      image 'docker:24-dind'            
      args  '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  environment {
    IMAGE_NAME         = "miUsuarioDocker/simple-java-maven-app"
    REGISTRY_CREDENTIAL = 'dockerhub-creds'
  }
  stages {
    stage('Build JAR') {
      steps {
        sh 'mvn clean package'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
      }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: REGISTRY_CREDENTIAL,
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
          sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
        }
      }
    }
    stage('Archive Artifacts') {
      steps {
        archiveArtifacts artifacts: 'target/*.jar'
      }
    }
  }
  post {
    success { slackSend color: 'good', message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} Image pushed" }
    failure { slackSend color: 'danger', message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED" }
  }
}
