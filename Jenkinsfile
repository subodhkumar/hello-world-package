pipeline {
    agent any 

    environment {
        NPM_AUTH_TOKEN = credentials('NPM_AUTH_TOKEN')
        GITHUB_PAT = credentials('GITHUB_PAT')
    }

    parameters {
        choice(name: 'VERSION_BUMP', choices: ['patch', 'minor', 'major'], description: 'Version bump type')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'  // More reliable than `npm install`
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'  // Ensure package works before publishing
            }
        }

        stage('Set Authentication') {
            steps {
                script {
                    sh '''
                        #!/bin/bash
                        set -e

                        echo "//registry.npmjs.org/:_authToken=${NPM_AUTH_TOKEN}" > ~/.npmrc
                        git config user.email "subodhkumarjc@gmail.com"
                        git config user.name "subodhkumar"
                        git remote set-url origin https://${GITHUB_PAT}@github.com/subodhkumar/hello-world-package.git
                    '''
                }
            }
        }

        stage('Bump Version & Publish') {
            steps {
                script {
                    def versionBump = params.VERSION_BUMP
                    def newVersion = sh(script: "npm version ${versionBump} --no-git-tag-version", returnStdout: true).trim()
                    
                    sh '''
                        #!/bin/bash
                        set -e

                        # Commit version bump before switching branches
                        git add package.json package-lock.json
                        git commit -m "Bump version before branch switch"

                        # Ensure we're on the correct branch
                        git rev-parse --abbrev-ref HEAD | grep -q '^main$' || git checkout main
                        git pull --rebase origin main || git rebase --abort

                        # Publish the package first
                        npm whoami
                        npm publish --access public
                    '''

                    // Only commit & push if publish succeeds
                    sh """
                        git push origin main
                        git push origin ${newVersion}
                    """

                }
            }
        }
    }

    post {
        success {
            echo '✅ Release successful!'
        }
        failure {
            echo '❌ Build failed! Check logs for details.'
        }
        always {
            sh 'rm -f ~/.npmrc'  // Cleanup credentials securely
        }
    }
}
