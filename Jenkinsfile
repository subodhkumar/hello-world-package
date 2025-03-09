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
                sh 'npm install'
            }
        }

        stage('Run Package') {
            steps {
                sh 'npm run start'
            }
        }

        stage('Set Authentication') {
            steps {
                script {
                    sh '''
                        #!/bin/bash
                        set -e

                        npm config set //registry.npmjs.org/:_authToken=$NPM_AUTH_TOKEN
                        git config user.email "subodhkumarjc@gmail.com"
                        git config user.name "subodhkumar"
                        git remote set-url origin https://$GITHUB_PAT@github.com/subodhkumar/hello-world-package.git
                    '''
                }
            }
        }

        stage('Bump Version, Commit & Publish') {
            steps {
                script {
                    def versionBump = params.VERSION_BUMP
                    sh """
                        #!/bin/bash
                        set -e

                        # Ensure we're on the main branch
                        git checkout main
                        git pull --rebase origin main

                        # Bump version and commit the change
                        npm version $versionBump -m "Bump version to %s [skip ci]"
                        git add package.json

                        # Push the commit and tag
                        git push origin main
                        git push origin --tags

                        # Publish the package to NPM
                        npm publish --access public
                    """
                }
            }
        }

        stage('Post Publish Cleanup') {
            steps {
                sh 'npm config delete //registry.npmjs.org/:_authToken'
            }
        }
    }
}
