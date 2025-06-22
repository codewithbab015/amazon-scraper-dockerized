pipeline {
    agent any

    environment {
        // Authentication and Git source
        GIT_URL = 'https://github.com/codewithbab015/amazon-scraper-dockerized.git'
        GIT_CREDENTIALS_ID = 'github_token_id'
        DOCKERHUB_CREDENTIALS_ID = 'docker_token_id'

        // Python env and cache files
        PIP_CACHE_DIR = "${WORKSPACE}/.pip_cache"
        VENV_DIR = "${WORKSPACE}/.venv"

        // Taskfile arguments
        RUN_GROUP = "electronics"
        RUN_NAME = "camera-photo"
        MAX_PAGE = "1"
        DESTINATION = "dir"
        LIMIT_RECORDS = "2"
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

        stage('Test') {
            steps {
                sh '''#!/bin/bash
                    set -e
                    echo "📄 Viewing Taskfile jobs..."
                    source "$VENV_DIR/bin/activate"

                    task default

                    echo "🧪 Starting ETL test runs..."
                    task cli-runner:run-jobs \
                        MAX=$MAX_PAGE \
                        DESTINATION=$DESTINATION \
                        RUN_GROUP=$RUN_GROUP \
                        RUN_NAME=$RUN_NAME
                '''
            }
        }
    }

    post {
        always {
            echo '📦 Archiving pip cache directory for reuse...'
            archiveArtifacts artifacts: '.pip_cache/**', fingerprint: true
        }
    }
}


    //     stage('DockerHub Login & Image Pull') {
    //         steps {
    //             withCredentials([usernamePassword(credentialsId: 
    //             "${env.DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
    //                 sh '''
    //                     echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
    //                 '''
    //             }

    //             sh '''
    //                 #!/bin/bash
    //                 set -e
    //                 docker pull mrbaloyin/etl-amazon-scraper-cli:latest
    //                 docker images
    //                 docker ps
    //             '''
    //         }
    //     }

    //     stage('Deploy Docker Img') {
    //         steps {
    //             echo 'All stages completed successfully.'
    //             echo 'You can now access the results in the workspace.'
    //             sh '''
    //                 #!/bin/bash
    //                 set -e
    //                 ls -lh
    //             '''
    //         }
    //     }
    // }
