pipeline {
    agent any

    environment {
        VENV_DIR = "venv"
        FLASK_APP = "app.py"
        FLASK_PORT = "5000"  // Port number your Flask app will run on
    }

    stages {
        stage('Setup Python Environment') {
            steps {
                sh '''
                    # Create virtual environment if not exist
                    if [ ! -d "$VENV_DIR" ]; then
                        python3 -m venv $VENV_DIR
                    fi
                    source $VENV_DIR/bin/activate
                    pip install --upgrade pip
                    pip install flask
                '''
            }
        }

        stage('Stop Existing Flask App') {
            steps {
                sh '''
                    pid=$(pgrep -f "$FLASK_APP" || true)
                    if [ ! -z "$pid" ]; then
                        echo "Stopping existing Flask app with PID: $pid"
                        kill -9 $pid
                    else
                        echo "No existing Flask app process found."
                    fi
                '''
            }
        }

        stage('Run Flask App') {
            steps {
                sh '''
                    source $VENV_DIR/bin/activate
                    nohup python $FLASK_APP > flask_app.log 2>&1 &
                    echo "Flask app started in background."
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully."
        }
        failure {
            echo "Deployment failed."
        }
    }
}
