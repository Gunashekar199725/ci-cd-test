pipeline {
    agent any

    environment {
        FLASK_APP = 'app.py'
        FLASK_ENV = 'production'
    }

    stages {
        stage('Setup Python Environment') {
            steps {
                script {
                    // Install Python 3 and pip if not already installed
                    sh '''
                        sudo apt update
                        sudo apt install -y python3 python3-pip
                    '''
                    // Install Flask
                    sh 'pip3 install flask'
                }
            }
        }

        stage('Stop Existing Flask App') {
            steps {
                script {
                    // Stop any running Flask application on port 5000
                    sh '''
                        pid=$(lsof -t -i:5000)
                        if [ -n "$pid" ]; then
                            kill -9 $pid
                        fi
                    '''
                }
            }
        }

        stage('Run Flask App in Background') {
            steps {
                script {
                    // Run the Flask application in the background
                    sh '''
                        nohup python3 $FLASK_APP > flask_app.log 2>&1 &
                    '''
                }
            }
        }

        stage('Verify Flask App') {
            steps {
                script {
                    // Check if the Flask application is running
                    sh '''
                        if ! pgrep -f $FLASK_APP > /dev/null; then
                            echo "Flask app failed to start"
                            exit 1
                        else
                            echo "Flask app is running"
                        fi
                    '''
                }
            }
        }
    }

    post {
        always {
            // Clean up any background processes if needed
            sh 'pkill -f $FLASK_APP || true'
        }
    }
}
