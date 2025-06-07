pipeline {
    agent any

    environment {
        IMAGE_NAME = "ci-cd-test"
        CONTAINER_NAME = "ci-cd-test"
        APP_DIR = "ci-cd-test"
        GIT_REPO = "https://github.com/Gunashekar199725/ci-cd-test.git"
        GIT_BRANCH = "main"
    }

    stages {
        stage('Prepare Directory') {
            steps {
                sh '''
                    rm -rf $APP_DIR
                    mkdir -p $APP_DIR
                '''
            }
        }

        stage('Clone Repository') {
            steps {
                dir("${APP_DIR}") {
                    git branch: "${GIT_BRANCH}", credentialsId: 'git_key', url: "${GIT_REPO}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir("${APP_DIR}") {
                    sh 'docker build -t $IMAGE_NAME .'
                }
            }
        }

        stage('Run the Container') {
            steps {
                sh """
                    echo "Stopping and removing any existing container..."
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    echo "Starting new container..."
                    docker run -d \\
                        --name ${CONTAINER_NAME} \\
                        --restart always \\
                        -p 8501:8501 \\
                        ${IMAGE_NAME}

                    echo "Container ${CONTAINER_NAME} is now running."
                """
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Deployment successful!'
        }
    }
}
