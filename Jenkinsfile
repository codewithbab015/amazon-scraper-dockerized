pipeline {
    agent any

    environment {
        // Auth and Git source
        GIT_URL = 'https://github.com/codewithbab015/amazon-scraper-dockerized.git'
        GIT_CREDENTIALS_ID = 'github_token_id'
        DOCKERHUB_CREDENTIALS_ID = 'docker_token_id'

        // Python env
        PIP_CACHE_DIR = "${WORKSPACE}/.pip_cache"
        VENV_DIR = "${WORKSPACE}/.venv"

        // Taskfile args
        RUN_GROUP = "electronics"
        RUN_NAME = "camera-photo"
        MAX_PAGE = "1"
        DESTINATION = "dir"
        LIMIT_RECORDS = "2"

        // Docker image metadata
        DOCKER_USER = "mrbaloyin"
        DOCKER_IMG = "amazon-web-scraper-cli"
    }

    stages {
        stage('Clone Source-Code') {
            steps {
                echo '📥 Cloning source code from Git...'
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: "${env.GIT_CREDENTIALS_ID}",
                        url: "${env.GIT_URL}"
                    ]]
                )
                echo '✅ Repository successfully cloned.'
            }
        }

        stage('Python Env-Setup') {
            steps {
                sh '''
                    echo "🔧 Setting up Python virtual environment..."
                    set -e
                    chmod +x activate_venv_ci.sh
                    ./activate_venv_ci.sh
                    echo "✅ Python environment setup complete."
                '''
            }
        }

        // stage('Test') {
        //     steps {
        //         sh '''#!/bin/bash
        //             set -e
        //             echo "📄 Viewing Taskfile jobs..."
        //             source "$VENV_DIR/bin/activate"
        //             task default

        //             echo "🧪 Starting ETL test runs..."
        //             task cli-runner:run-jobs \
        //                 MAX=$MAX_PAGE \
        //                 DESTINATION=$DESTINATION \
        //                 RUN_GROUP=$RUN_GROUP \
        //                 RUN_NAME=$RUN_NAME > task_run.log 2>&1
        //         '''
        //     }
        // }

        stage('Build') {
            steps {
                script {
                    // Set Git SHA and final image tag
                    def gitSha = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.DOCKER_TAG = "${env.DOCKER_USER}/${env.DOCKER_IMG}:${gitSha}"

                    // Docker Hub login
                    def dockerLoginFunc = {
                        withCredentials([usernamePassword(
                            credentialsId: env.DOCKERHUB_CREDENTIALS_ID,
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh '''
                                echo "🔐 Logging into Docker Hub..."
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            '''
                        }
                    }

                    dockerLoginFunc()

                    // Build and push image
                    sh '''
                        set -e
                        echo "🐳 Building Docker image..."
                        docker buildx build -t $DOCKER_TAG .

                        echo "📤 Pushing Docker image to Docker Hub..."
                        docker push $DOCKER_TAG

                        echo "🆕 Also tagging as 'latest'..."
                        docker tag $DOCKER_TAG $DOCKER_USER/$DOCKER_IMG:latest
                        docker push $DOCKER_USER/$DOCKER_IMG:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            echo '📦 Archiving logs and pip cache...'
            archiveArtifacts artifacts: '.pip_cache/**', fingerprint: true
            archiveArtifacts artifacts: 'task_run.log', fingerprint: true
        }
    }
}
