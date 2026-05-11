pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify PHP & PHPUnit') {
            steps {
                sh 'php --version'
                sh 'phpunit --version'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'phpunit --bootstrap src/OrderProcessor.php tests/'
            }
        }
    }

    post {
        success {
            echo '✅ All tests passed — build successful!'
        }
        failure {
            echo '❌ Build failed — unit test errors above.'
        }
    }
}
