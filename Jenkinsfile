pipeline {
    agent {
        docker {
            image 'node:18'
        }
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Run Package') {
            steps {
                sh 'npm run start'
            }
        }
    }
}