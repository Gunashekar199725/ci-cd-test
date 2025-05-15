pipeline {
    agent any

    environment {
        VENV = "venv"
        PORT = "5000"
    }

    stages {
        stage('Setup Python & Flask') {
            steps {
                sh '''
                    echo "[INFO] Setting up virtual environment..."
                    python3 -m venv ${VENV}
                    . ${VENV}/bin/activate
                    pip install --upgrade pip
                    pip install flask
                '''
            }
        }

        stage('Stop Existing Flask App') {
            steps {
                sh '''
                    echo "[INFO] Stopping previous Flask process if exists..."
                    if [ -f flask.pid ]; then
                        PID=$(cat flask.pid)
                        if kill -0 $PID > /dev/null 2>&1; then
                            kill -9 $PID || true
                        fi
                        rm -f flask.pid
                    fi
                '''
            }
        }

        stage('Run Flask App in Background') {
            steps {
                sh '''
                    echo "[INFO] Starting Flask app on 0.0.0.0:${PORT}..."
                    . ${VENV}/bin/activate
                    nohup python3 app.py > flask.log 2>&1 &
                    echo $! > flask.pid
                    sleep 5
                '''
            }
        }

        stage('Verify Flask Process') {
            steps {
                sh '''
                    if [ -f flask.pid ]; then
                        PID=$(cat flask.pid)
                        if ps -p $PID > /dev/null 2>&1; then
                            echo "[INFO] Flask app running with PID $PID"
                        else
                            echo "[ERROR] Flask app process not found"
                            exit 1
                        fi
                    else
                        echo "[ERROR] flask.pid not found"
                        exit 1
                    fi
                '''
            }
        }

        stage('Verify Internal Access') {
            steps {
                sh '''
                    echo "[INFO] Testing app on localhost:${PORT}..."
                    curl -s http://localhost:${PORT} || (echo "[WARN] App not responding yet" && exit 1)
                '''
            }
        }
    }
}
