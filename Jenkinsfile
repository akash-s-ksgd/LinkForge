pipeline {
    agent any

    // Automatic triggering: GitHub webhook (githubPush) with SCM polling as fallback
    triggers {
        githubPush()
        pollSCM('H/2 * * * *')
    }

    environment {
        IMAGE_NAME = 'smart-url-shortener'
        VENV       = '.venv'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment Check') {
            steps {
                bat 'node -v'
                bat 'npm -v'
                bat 'git --version'
                bat 'python --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Lint') {
            steps {
                bat 'npm run lint'
            }
        }

        stage('Build') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'dist/*', fingerprint: true
            }
        }

        stage('Deploy to GitHub Pages') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-pat', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        bat '''
                            if exist dist\\* (
                                git checkout --orphan gh-pages
                                git rm -rf . >nul 2>&1
                                xcopy /Y /Q dist\\* .
                                git add .
                                git commit -m "Deploy build ${BUILD_NUMBER}"
                                git push origin gh-pages --force
                            )
                        '''
                    }
                }
            }
        }
    }

    post {
        success { echo "Build ${env.BUILD_NUMBER} succeeded" }
        failure { echo "Build ${env.BUILD_NUMBER} failed" }
    }
}
