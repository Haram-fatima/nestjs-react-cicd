pipeline {
    agent any

    environment {
        BACKEND_DIR = 'backend'
        FRONTEND_DIR = 'frontend'
        SERVER_USER = 'youruser'
        SERVER_IP = 'your.server.ip'
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
                    sh 'npm install'
                    sh 'npm test || true'
                    sh 'npm run build'
                    sh 'tar -czf backend.tar.gz dist'
                    stash includes: 'backend/backend.tar.gz', name: 'backend'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'npm install'
                    sh 'npm test || true'
                    sh 'npm run build'
                    sh 'tar -czf frontend.tar.gz build'
                    stash includes: 'frontend/frontend.tar.gz', name: 'frontend'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                unstash 'backend'
                sh '''
                    scp backend/backend.tar.gz $SERVER_USER@$SERVER_IP:/tmp/
                    ssh $SERVER_USER@$SERVER_IP '
                        mkdir -p ~/rollback/backend_$(date +%F-%T) &&
                        cp -r ${BACKEND_PATH} ~/rollback/backend_$(date +%F-%T) || true &&
                        tar -xzf /tmp/backend.tar.gz -C ${BACKEND_PATH} &&
                        cd ${BACKEND_PATH} &&
                        npm install --omit=dev &&
                        pm2 restart app || pm2 start dist/main.js --name app
                    '
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                unstash 'frontend'
                sh '''
                    scp frontend/frontend.tar.gz $SERVER_USER@$SERVER_IP:/tmp/
                    ssh $SERVER_USER@$SERVER_IP '
                        mkdir -p ~/rollback/frontend_$(date +%F-%T) &&
                        cp -r ${FRONTEND_PATH} ~/rollback/frontend_$(date +%F-%T) || true &&
                        rm -rf ${FRONTEND_PATH}/* &&
                        tar -xzf /tmp/frontend.tar.gz -C ${FRONTEND_PATH} &&
                        nginx -s reload
                    '
                '''
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

