
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t portfolio-devops:jenkins-%BUILD_NUMBER% .'
            }
        }

        stage('Verify Docker Image') {
            steps {
                bat 'docker image inspect portfolio-devops:jenkins-%BUILD_NUMBER% >NUL'
            }
        }

        stage('Deploy Docker Container') {
            steps {
                // Remove old test container if present
                bat(
                    script: 'docker rm -f portfolio-jenkins >NUL 2>&1',
                    returnStatus: true
                )

                // Deploy latest image
                bat 'docker run -d --name portfolio-jenkins -p 8086:80 portfolio-devops:jenkins-%BUILD_NUMBER%'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'curl.exe --fail --retry 5 --retry-delay 2 --retry-connrefused -I http://127.0.0.1:8086/'
            }
        }
    }
}
