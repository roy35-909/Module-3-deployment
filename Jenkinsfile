pipeline {
    agent any

    triggers {
        // Trigger on push to main branch (requires webhook setup in Git)
        githubPush()
    }

    environment {
        NODE_VERSION = '18'
        APP_NAME = 'node-express-app'
        GIT_REPO = 'https://github.com/YOUR_USERNAME/Module-3-deployment.git'
        DEPLOY_DIR = '/var/www/app'
    }

    tools {
        // Use NodeJS plugin - make sure it's configured in Jenkins Global Tool Configuration
        nodejs "${NODE_VERSION}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling or cloning the repository...'
                checkout scm
                // Or use git clone explicitly:
                // git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Install Node.js') {
            steps {
                echo 'Verifying Node.js installation...'
                script {
                    // Check if Node.js is available
                    def nodeExists = sh(script: 'which node', returnStatus: true) == 0
                    if (!nodeExists) {
                        echo 'Node.js not found. Please ensure NodeJS plugin is installed and configured.'
                        error('Node.js is required but not available')
                    }
                }
                sh 'node --version'
                sh 'npm --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing npm dependencies...'
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running npm run check...'
                sh 'npm run check'
            }
        }

        stage('Deploy with PM2') {
            steps {
                echo 'Deploying application with PM2...'
                script {
                    // Check if PM2 is installed globally, install if not
                    def pm2Exists = sh(script: 'which pm2', returnStatus: true) == 0
                    if (!pm2Exists) {
                        echo 'Installing PM2 globally...'
                        sh 'npm install -g pm2'
                    }

                    // Stop existing app if running (ignore errors if not running)
                    sh "pm2 stop ${APP_NAME} || true"
                    sh "pm2 delete ${APP_NAME} || true"

                    // Start the application with PM2
                    sh "pm2 start src/server.js --name ${APP_NAME}"

                    // Save PM2 process list for restart on reboot
                    sh 'pm2 save'

                    // Show PM2 status
                    sh 'pm2 status'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully! Application deployed with PM2.'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
        always {
           
            cleanWs()
        }
    }
}
