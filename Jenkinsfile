pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                echo "Clone Respository"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'docker-compose run composer install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'docker-compose run laravel php artisan test'
            }
        }
    }

    post {
        success {
            echo 'Build and tests succeeded!'
        }
        failure {
            echo 'Build or tests failed!'
        }
    }
}
