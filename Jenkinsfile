pipeline {
  /* 1) Arranca TODO el Pipeline dentro del contenedor maven:3.8.8-openjdk-17 */
  agent {
    docker {
      image 'maven:3.8.8-openjdk-17'
      args  '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    IMAGE_NAME = 'codefher/spring-web-service'
    SERVICE_PORT = '8081'  // si tu app arranca en 8081
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare') {
      steps {
        sh 'chmod +x mvnw'
      }
    }

    stage('Build & Test') {
      steps {
        // Dentro del contenedor ya hay Java y Maven, así que ./mvnw funciona perfectamente
        sh './mvnw clean package -DskipTests'
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
      echo "❌ Falló el pipeline, revisa los logs y el service.log"
    }
  }
}
