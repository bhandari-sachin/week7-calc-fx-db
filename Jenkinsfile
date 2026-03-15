pipeline {
    agent any

    environment {
        IMAGE_NAME = "calculator-fx-db"
        IMAGE_TAG  = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Compiling with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Compose Up') {
            steps {
                echo 'Starting services (MariaDB + Calculator)...'
                sh 'docker compose up -d --build'
            }
        }

        stage('Verify Database') {
            steps {
                echo 'Waiting for MariaDB to be ready...'
                // Give MariaDB a few seconds after healthcheck
                sh 'sleep 10'
                sh '''
                    docker exec calculator_db mariadb \
                        -u calcuser -pcalcpass calculatordb \
                        -e "SHOW TABLES; SELECT * FROM calculations;"
                '''
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Stopping containers...'
                sh 'docker compose down'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs above.'
            sh 'docker compose down || true'
        }
    }
}