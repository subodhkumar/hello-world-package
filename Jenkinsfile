pipeline {
    agent any
    environment {
        NPM_AUTH_TOKEN = credentials('NPM_AUTH_TOKEN')
        GITHUB_PAT = credentials('GITHUB_PAT')
    }
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
        stage('Set authentication') {
            steps {
                sh 'npm config set //registry.npmjs.org/:_authToken=${NPM_AUTH_TOKEN}'
                sh """
                    git config --global user.email "subodhkumarjc@gmail.com"
                    git config --global user.name "Jenkins CI"
                    git config set-url origin https://${GITHUB_PAT}@github.com/subodhkumar/hello-world-package.git
                """
            }
        }
        stage('Update Version'){
            steps {
                sh 'npm version ${params.VERSION_BUMP} --no-commit-hooks --no-git-tag-version'
            }
        }
        stage('Commit & Push Version Update'){
            steps {
                script {
                    env.NEW_VERSION = sh(script: "node -p \"require('./package.json').version\"", returnStdout: true).trim();
                    sh """
                        git add package.json
                        git commit -m "Bump version to ${env.NEW_VERSION}"
                        git tag "v${env.NEW_VERSION}"
                        git push origin HEAD:master --tags
                    """
                }
            }
        }
        stage('Post Publish cleanup'){
            steps {
                sh 'npm config delete //registry.npmjs.org/:_authToken'
            }
        }
    }
}