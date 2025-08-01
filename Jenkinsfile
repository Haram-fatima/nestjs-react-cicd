pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
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
                    stash includes: 'backend.tar.gz', name: 'backend'
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
                    stash includes: 'frontend.tar.gz', name: 'frontend'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                unstash 'backend'
                sh '''
                    scp backend.tar.gz ${SERVER_USER}@${SERVER_IP}:/tmp/
                    ssh ${SERVER_USER}@${SERVER_IP} <<EOF
                        TIMESTAMP=$(date +%F-%T)
                        mkdir -p ~/rollback/backend_$TIMESTAMP
                        cp -r ${BACKEND_PATH}/* ~/rollback/backend_$TIMESTAMP/ || true
                        rm -rf ${BACKEND_PATH}/*
                        mkdir -p ${BACKEND_PATH}
                        tar -xzf /tmp/backend.tar.gz -C ${BACKEND_PATH}
                        cd ${BACKEND_PATH}
                        npm install --omit=dev
                        pm2 restart app || pm2 start dist/main.js --name app
                    EOF
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                unstash 'frontend'
                sh '''
                    scp frontend.tar.gz ${SERVER_USER}@${SERVER_IP}:/tmp/
                    ssh ${SERVER_USER}@${SERVER_IP} <<EOF
                        TIMESTAMP=$(date +%F-%T)
                        mkdir -p ~/rollback/frontend_$TIMESTAMP
                        cp -r ${FRONTEND_PATH}/* ~/rollback/frontend_$TIMESTAMP/ || true
                        rm -rf ${FRONTEND_PATH}/*
                        mkdir -p ${FRONTEND_PATH}
                        tar -xzf /tmp/frontend.tar.gz -C ${FRONTEND_PATH}
                        nginx -s reload
                    EOF
                '''
            }
        }

        stage('Health Check') {
            steps {
                script {
                    def backendUrl = "http://${SERVER_IP}:3000/health"
                    def frontendUrl = "http://${SERVER_IP}"

                    def backendResponse = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" ${backendUrl}", returnStdout: true).trim()
                    def frontendResponse = sh(script: "curl -s -o /dev/null -w \"%{http_code}\" ${frontendUrl}", returnStdout: true).trim()

                    if (backendResponse != "200" || frontendResponse != "200") {
                        error("Health check failed! Backend: ${backendResponse}, Frontend: ${frontendResponse}")
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful'
        }
        failure {
            echo 'Deployment failed. Rollback may be required.'
        }
    }
}

        
