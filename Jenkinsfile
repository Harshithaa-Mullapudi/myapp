pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhubusername/myapp"
        DOCKER_TAG = "v${env.BUILD_NUMBER}"
        DEPLOY_SERVER = "user@your-target-server"
        SSH_CRED = "deploy-key"
        DOCKER_CRED = "dockerhub-creds"
        SONARQUBE_ENV = "MySonarQubeServer"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/yourusername/myapp.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    if (fileExists('package.json')) {
                        sh 'npm install'
                        sh 'npm run build || true'
                    } else if (fileExists('requirements.txt')) {
                        sh 'pip install -r requirements.txt'
                    } else if (fileExists('pom.xml')) {
                        sh 'mvn clean package -DskipTests'
                    }
                }
            }
        }

        stage('Gitleaks Scan') {
            steps {
                sh 'docker run --rm -v $PWD:/repo zricethezav/gitleaks:latest detect --source=/repo --no-banner || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    """
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh """
                docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ["${SSH_CRED}"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                        docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                        cd /opt/myapp
                        sed -i "s|image: .*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|" docker-compose.yml
                        docker compose up -d
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            emailext subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "Build and deployment succeeded!",
                     to: "team@example.com"
        }
        failure {
            emailext subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "Build failed. Check Jenkins logs.",
                     to: "team@example.com"
        }
    }
}

