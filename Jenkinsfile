pipeline {
    agent any
    parameters {
        choice(name: 'VERSION_BUMP', choices:['patch','minor','major'], description: 'Version bump type');
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