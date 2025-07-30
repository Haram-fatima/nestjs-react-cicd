
pipeline {
    agent any

    tools {
        nodejs 'NodeJS'  // ✅ Matches Global Tool Configuration
    }

    environment {
        BACKEND_DIR = 'backend'
        FRONTEND_DIR = 'frontend'
        SERVER_USER = 'jenkins'         
        SERVER_IP = '10.171.221.161'    
        BACKEND_PATH = '/var/www/backend'
        FRONTEND_PATH = '/var/www/frontend'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh '''
                        npm install
                        npm test || true
                        npm run build
                        tar -czf backend.tar.gz dist
                    '''
                    stash includes: 'dist/**', name: 'backend'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh '''
                        npm install
                        npm test || true
                        npm run build
                        tar -czf frontend.tar.gz build
                    '''
                    stash includes: 'build/**', name: 'frontend'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                unstash 'backend'
                sh '''
                    scp backend.tar.gz jenkins@10.171.221.161:/tmp/
                    ssh jenkins@10.171.221.161 '
                        mkdir -p ~/rollback/backend_$(date +%F-%T)
                        cp -r ${BACKEND_PATH} ~/rollback/backend_$(date +%F-%T) || true
                        rm -rf ${BACKEND_PATH}/*
                        mkdir -p ${BACKEND_PATH}
                        tar -xzf /tmp/backend.tar.gz -C ${BACKEND_PATH}
                        cd ${BACKEND_PATH}
                        npm install --omit=dev
                        pm2 restart app || pm2 start dist/main.js --name app
                    '
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                unstash 'frontend'
                sh '''
                    scp frontend.tar.gz jenkins@10.171.221.161:/tmp/
                    ssh jenkins@10.171.221.161 '
                        mkdir -p ~/rollback/frontend_$(date +%F-%T)
                        cp -r ${FRONTEND_PATH} ~/rollback/frontend_$(date +%F-%T) || true
                        rm -rf ${FRONTEND_PATH}/*
                        mkdir -p ${FRONTEND_PATH}
                        tar -xzf /tmp/frontend.tar.gz -C ${FRONTEND_PATH}
                        nginx -s reload
                    '
                '''
            }
        }

        stage('Health Check') {
            steps {
                script {
                    def backendUrl = "http://${SERVER_IP}:3000/health"  // ✅ Adjust this if your API uses a different path
                    def frontendUrl = "http://${SERVER_IP}"             // Nginx hosted frontend

                    echo "🔍 Checking backend health..."
                    def backendResponse = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" ${backendUrl}", returnStdout: true).trim()

                    echo "🔍 Checking frontend health..."
                    def frontendResponse = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" ${frontendUrl}", returnStdout: true).trim()

                    if (backendResponse != "200" || frontendResponse != "200") {
                        error("❌ Health check failed! Backend: ${backendResponse}, Frontend: ${frontendResponse}")
                    }

                    echo "✅ Health check passed! Backend: ${backendResponse}, Frontend: ${frontendResponse}"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful'
        }
        failure {
            echo '❌ Deployment failed. Rollback may be required.'
        }
    }
}
