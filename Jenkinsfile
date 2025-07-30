pipeline {
    agent any

    tools {
        nodejs 'NodeJS'  // ✅ Make sure this matches Global Tool Configuration exactly
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
                    scp backend.tar.gz ${SERVER_USER}@${SERVER_IP}:/tmp/
                    ssh ${SERVER_USER}@${SERVER_IP} '
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
                    scp frontend.tar.gz ${SERVER_USER}@${SERVER_IP}:/tmp/
                    ssh ${SERVER_USER}@${SERVER_IP} '
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
    }

    post {
        success {
            echo '✅ Deployment successful'

            archiveArtifacts artifacts: '**/backend.tar.gz, **/frontend.tar.gz', fingerprint: true

            emailext (
                subject: "✅ SUCCESS: Jenkins Build #${env.BUILD_NUMBER}",
                body: "Project deployed successfully.\n\nCheck build: ${env.BUILD_URL}",
                to: "youremail@example.com"
            )

            // slackSend (optional if Slack configured)
            // slackSend channel: '#devops', color: 'good', message: "✅ SUCCESS: Jenkins Job ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
        }

        failure {
            echo '❌ Deployment failed. Rollback may be required.'

            emailext (
                subject: "❌ FAILURE: Jenkins Build #${env.BUILD_NUMBER}",
                body: "Deployment failed. Please check the logs.\n\nBuild URL: ${env.BUILD_URL}",
                to: "youremail@example.com"
            )

            // slackSend (optional if Slack configured)
            // slackSend channel: '#devops', color: 'danger', message: "❌ FAILED: Jenkins Job ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
        }

        always {
            echo "📦 Job complete: ${currentBuild.result}"
        }
    }
}
