pipeline {
    agent any

    // Use Node.js installed from Manage Jenkins → Tools
    tools {
        nodejs 'NodeJS 18.x'   // 👈 match the name you configured
    }

    environment {
        // 👇 Match the names you set under "Manage Jenkins → Configure System"
        SONARQUBE = 'MySonarQubeServer'
        DOCKER_HUB_CREDS = 'dockerhub-credentials-id'  // Jenkins credential ID
        EMAIL_TO = 'mullapudiharshitha2003@gmail.com'    // change this to your email
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Harshithaa-Mullapudi/myapp'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build App') {
            steps {
                sh 'npm run build || echo "No build step defined"'
            }
        }

        stage('SonarQube Scan') {
            environment {
                scannerHome = tool 'sonar-scanner' // 👈 match the name from Tool Config
            }
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh '''
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=TaskManager \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerhub-credentials-id",
                                                usernameVariable: 'DOCKER_USER',
                                                passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag myapp:latest $DOCKER_USER/myapp:latest
                        docker push $DOCKER_USER/myapp:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Deploying application to target server...'
                // You can later add SSH commands here
            }
        }
    }

    post {
        success {
            emailext subject: "✅ SUCCESS: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                     body: "Build successful!\n\nSee details at ${env.BUILD_URL}",
                     to: "${EMAIL_TO}"
        }
        failure {
            emailext subject: "❌ FAILURE: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                     body: "Build failed.\n\nCheck logs at ${env.BUILD_URL}",
                     to: "${EMAIL_TO}"
        }
    }
}
