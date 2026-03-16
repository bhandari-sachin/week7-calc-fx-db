
pipeline {
  agent any

  environment {
    PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;${env.PATH}"
    DOCKERHUB_CREDENTIALS_ID = '11de06b8-c29b-4e4c-bf92-2d6a8d92868e'
    DOCKERHUB_REPO = 'sachinbhandari/calculator-fx-db'
    DOCKER_IMAGE_TAG = 'latest'
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/bhandari-sachin/week7-calc-fx-db.git'
      }
    }

    stage('Build') {
      steps {
        bat 'mvn clean install -DskipTests'
      }
    }

    stage('Test') {
      steps {
        bat 'mvn test'
      }
    }

    stage('Report') {
      steps {
        bat 'mvn jacoco:report'
      }
    }

    stage('Publish Test Results') {
      steps {
        junit '**/target/surefire-reports/*.xml'
      }
    }

    stage('Publish Coverage Report') {
      steps {
        jacoco()
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}")
        }
      }
    }

    stage('Push Docker Image to Docker Hub') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS_ID) {
            docker.image("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}").push()
          }
        }
      }
    }

    stage('Deploy with Docker Compose') {
      steps {
        script {
          bat """
            set DOCKERHUB_USERNAME=sachinbhandari
            set APP_IMAGE_TAG=${DOCKER_IMAGE_TAG}
            docker compose pull calculator
            docker compose up -d --force-recreate
          """
        }
      }
    }

    stage('Verify Database') {
      steps {
        bat 'ping -n 16 127.0.0.1 > nul'
        bat 'docker exec calculator_db mariadb -u calcuser -pcalcpass calculatordb -e "SHOW TABLES; SELECT * FROM calculations;"'
      }
    }
  }

  post {
    success {
      echo "Pipeline succeeded. Image pushed: ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}"
    }

    failure {
      echo 'Pipeline failed. Check logs above.'
      bat 'docker compose down || exit 0'
    }

    cleanup {
      bat 'docker image prune -f || exit 0'
    }
  }
}

