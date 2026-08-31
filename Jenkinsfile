pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *') // Checks GitHub for new commits every 5 minutes
    }

    tools {
        maven 'Maven-3.9' // Must match the exact name configured in Manage Jenkins -> Tools
    }

    environment {
        IMAGE_NAME = "mywebsite"
        CONTAINER_NAME = "mywebsite-container"
        APP_PORT = "8081"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mani1987-87/my-website.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --name mywebsite-container --restart unless-stopped -p 8081:8080 mywebsite:latest
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh 'sleep 30'
                sh 'curl -f http://localhost:8081/actuator/health'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully. App is running on port 8081.'
        }
        failure {
            echo 'Pipeline failed. Check the logs above.'
        }
    }
}