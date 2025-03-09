pipeline {
    agent any // Runs the pipeline on any available Jenkins agent

    environment {
        NPM_AUTH_TOKEN = credentials('NPM_AUTH_TOKEN') // Fetch NPM authentication token from Jenkins credentials
        GITHUB_PAT = credentials('GITHUB_PAT') // Fetch GitHub Personal Access Token from Jenkins credentials
    }

    parameters {
        // User can choose how to bump the version: patch (1.0.1), minor (1.1.0), or major (2.0.0)
        choice(name: 'VERSION_BUMP', choices: ['patch', 'minor', 'major'], description: 'Version bump type')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm // Checks out the repository from source control
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install' // Installs all npm dependencies
            }
        }

        stage('Run Package') {
            steps {
                sh 'npm run start' // Runs the application (if applicable)
            }
        }

        stage('Set authentication') {
            steps {
                script {
                    sh '''
                        #!/bin/bash
                        set -e # Exit immediately if a command exits with a non-zero status
                        set -o pipefail # Ensures pipeline errors are properly handled

                        # Set npm authentication token for publishing
                        npm config set //registry.npmjs.org/:_authToken=$NPM_AUTH_TOKEN

                        # Configure Git with Jenkins bot details (avoids issues with commit ownership)
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins CI"

                        # Update GitHub remote URL to use the personal access token (PAT) for authentication
                        git remote set-url origin https://$GITHUB_PAT@github.com/subodhkumar/hello-world-package.git
                    '''
                }
            }
        }

        stage('Bump version, Commit & Publish') {
            steps {
                script {
                    sh '''
                        #!/bin/bash
                        set -e # Stop the script if any command fails
                        set -o pipefail # Catch errors in pipelines

                        # Read the user-provided version bump type (patch, minor, or major)
                        VERSION_BUMP=${params.VERSION_BUMP}

                        # Bump version in package.json and create a commit with a message
                        npm version $VERSION_BUMP -m "Bump version to %s [skip ci]"

                        # Get the currently checked-out branch name (to avoid forcing main)
                        CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

                        # Ensure package.json and package-lock.json changes are committed
                        git add package.json package-lock.json

                        # Amend the previous commit to include these changes (without creating a new commit)
                        git commit --amend --no-edit

                        # Push the commit and tags to the same branch Jenkins checked out
                        git push origin HEAD:$CURRENT_BRANCH --tags

                        # Publish the package to npm registry
                        npm publish --access public
                    '''
                }
            }
        }

        stage('Post Publish Cleanup') {
            steps {
                sh 'npm config delete //registry.npmjs.org/:_authToken' // Remove NPM authentication token from local config for security
            }
        }
    }
}
