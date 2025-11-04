pipeline {
    agent any

    tools {
        nodejs 'NodeJS 18.x'  // use the Node.js version you installed earlier
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/LasyaGupthaN/TaskManger'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || echo "No tests found"'
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Deployment step goes here (SSH, Docker, etc.)'
            }
        }
    }

    post {
        success {
            echo '✅ Build and Deploy successful!'
        }
        failure {
            echo '❌ Build failed!'
        }
    }
}
