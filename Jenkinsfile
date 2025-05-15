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
                    ${VENV}/bin/pip install --upgrade pip
                    ${VENV}/bin/pip install flask
                '''
            }
        }

        stage('Stop Existing Flask App') {
            steps {
                sh '''
                    echo "[INFO] Stopping previous Flask process if exists..."
                    if [ -f flask.pid ]; then
                        kill -9 $(cat flask.pid) || true
                        rm flask.pid
                        sleep 2
                    fi
                '''
            }
        }

        stage('Run Flask App in Background') {
            steps {
                sh '''
                    echo "[INFO] Starting Flask app on 0.0.0.0:${PORT}"
                    nohup ${VENV}/bin/python3 app.py > flask.log 2>&1 &
                    echo $! > flask.pid
                    sleep 2
                    echo "[INFO] Current running processes:"
                    ps -ef | grep app.py | grep -v grep || echo "[WARN] Flask app process not found"
                '''
            }
        }

        stage('Verify Internal Access') {
            steps {
                sh '''
                    sleep 3
                    echo "[INFO] Testing app on localhost:${PORT}"
                    if curl -s http://localhost:${PORT}; then
                        echo "[SUCCESS] App responded on port ${PORT}"
                    else
                        echo "[ERROR] App did not respond. Dumping log:"
                        cat flask.log
                        exit 1
                    fi
                '''
            }
        }
    }
}
