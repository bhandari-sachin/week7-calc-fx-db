
pipeline {
  agent any

  environment {
    PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;${env.PATH}"
    DOCKERHUB_CREDENTIALS_ID = '11de06b8-c29b-4e4c-bf92-2d6a8d92868e'
    DOCKERHUB_REPO = 'sachinbhandari/calculator-fx-db'
    DOCKER_IMAGE_TAG = 'latest'
    DOCKER_IMAGE_TAG_BUILD  = "${env.BUILD_NUMBER}"
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
      junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
    }
  }

  /*
  stage('Publish Coverage Report') {
    steps {
      jacoco()
    }
  }
  */

  stage('Test Docker') {
    steps {
      bat 'docker --version'
      bat 'docker info'
    }
  }

  stage('Build Docker Image') {
    steps {
      bat """
        docker build -t ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG} .
      """
    }
  }

        stage('Push Docker Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin

                        docker push %BACKEND_IMAGE_REPO%:%DOCKER_IMAGE_TAG_BUILD%
                        docker push %BACKEND_IMAGE_REPO%:%DOCKER_IMAGE_TAG_LATEST%

                        docker push %FRONTEND_IMAGE_REPO%:%DOCKER_IMAGE_TAG_BUILD%
                        docker push %FRONTEND_IMAGE_REPO%:%DOCKER_IMAGE_TAG_LATEST%

                        docker logout
                    """
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

