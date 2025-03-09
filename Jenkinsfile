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
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins CI"
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

                        // CURRENT_BRANCH=\$(git rev-parse --abbrev-ref HEAD)

                        # Ensure we have the latest changes
                        // git pull --rebase origin \$CURRENT_BRANCH

                        # Bump version and commit the change
                        git add package.json
                        npm version $versionBump -m "Bump version to %s [skip ci]"

                        # Push the commit and tag
                        git push origin HEAD:main --tags

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
