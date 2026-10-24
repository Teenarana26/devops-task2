pipeline {
    agent any

    environment {
        IMAGE_NAME     = 'jenkins-cicd-demo'
        CONTAINER_NAME = 'jenkins-cicd-demo'
        APP_PORT       = '3000'
    }

    // Trigger on each commit: GitHub webhook (instant) with SCM polling as a fallback
    triggers {
        githubPush()
        pollSCM('H/2 * * * *')
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker image...'
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% -t %IMAGE_NAME%:latest .'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests inside the container...'
                bat 'docker run --rm %IMAGE_NAME%:%BUILD_NUMBER% npm test'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying container...'
                bat '''
                    docker rm -f %CONTAINER_NAME%
                    docker run -d --name %CONTAINER_NAME% -p %APP_PORT%:3000 %IMAGE_NAME%:%BUILD_NUMBER%
                    exit /b 0
                '''
            }
        }

        stage('Verify') {
            steps {
                bat '''
                    ping -n 5 127.0.0.1 > nul
                    curl -f http://localhost:%APP_PORT%/health
                '''
            }
        }
    }

    post {
        success { echo "Build #${env.BUILD_NUMBER} deployed successfully." }
        failure { echo "Build #${env.BUILD_NUMBER} failed. Check the console output." }
    }
}

