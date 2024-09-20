pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/swizzen1/jenkins.git'
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
