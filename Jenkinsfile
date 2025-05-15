pipeline {
    agent any

    environment {
        FLASK_APP = 'app.py'
        VENV_DIR = 'venv'
    }

    stages {
        stage('Setup Python Environment') {
            steps {
                script {
                    // Install python3-venv package to create virtualenv
                    sh '''
                        sudo apt update
                        sudo apt install -y python3-venv
                        python3 -m venv $VENV_DIR
                        source $VENV_DIR/bin/activate
                        pip install --upgrade pip
                        pip install flask
                    '''
                }
            }
        }

        stage('Stop Existing Flask App') {
            steps {
                script {
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
                    // Run Flask app using the virtual environment python interpreter
                    sh '''
                        source $VENV_DIR/bin/activate
                        nohup python $FLASK_APP > flask_app.log 2>&1 &
                    '''
                }
            }
        }

        stage('Verify Flask App') {
            steps {
                script {
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
            // Optionally clean up Flask app process on job finish
            sh 'pkill -f $FLASK_APP || true'
        }
    }
}
