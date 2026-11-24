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
                retry(6) {
                    sleep time: 5, unit: 'SECONDS'
                    bat 'curl --fail http://localhost:9000/api/health'
                }
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
