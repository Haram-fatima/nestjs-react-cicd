pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Haram-fatima/nestjs-react-cicd.git'

            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                    sh 'npm run build'
                    sh 'docker build -t backend-image .'
                    sh 'docker save backend-image -o backend-image.tar'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                    sh 'docker build -t frontend-image .'
                    sh 'docker save frontend-image -o frontend-image.tar'
                }
            }
        }

        stage('Build Completed') {
            steps {
                echo '🎉 Build process completed successfully. Docker images for backend and frontend are ready.'
            }
        }
    }
}

