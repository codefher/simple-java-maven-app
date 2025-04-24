pipeline {
  agent {
    docker {
      image 'maven:3.8.8-openjdk-17'
      // Para poder llamar a docker build/push si lo necesitas:
      args  '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    IMAGE_NAME = 'codefher/spring-web-service'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare') {
      steps {
        // Asegura permisos
        sh 'chmod +x mvnw'
      }
    }

    stage('Build & Test') {
      steps {
        // Ahora mvnw encontrará JAVA_HOME dentro del contenedor maven:...-openjdk-17
        sh './mvnw clean package'
      }
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
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
            dockerImage.push()
          }
        }
      }
    }

    stage('Deploy to Staging') {
      steps {
        dir('deploy') {
          withEnv(["BUILD_NUMBER=${env.BUILD_NUMBER}"]) {
            sh 'docker-compose down || true'
            sh 'docker-compose pull'
            sh 'docker-compose up -d'
          }
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'service.log', allowEmptyArchive: true
    }
    success {
      echo "✅ Deployed ${IMAGE_NAME}:${env.BUILD_NUMBER}"
    }
    failure {
      echo "❌ Falló el pipeline, revisa los logs"
    }
  }
}
