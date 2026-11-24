pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                bat 'docker compose down'
                bat 'docker compose up -d'
            }
        }

        stage('Test') {
            steps {
                bat 'curl --fail --retry 10 --retry-delay 3 --retry-connrefused http://localhost:9000/api/health'
            }
        }
    }

    post {
        always {
            bat 'docker compose ps'
            bat 'docker compose down'
        }
    }
}
