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
                        kill -9 $(cat flask.pid) || true
                        rm flask.pid
                    fi
                '''
            }
        }

        stage('Run Flask App in Background') {
            steps {
                sh '''
                    echo "[INFO] Starting Flask app on 0.0.0.0:${PORT}"
                    . ${VENV}/bin/activate
                    nohup python3 app.py > flask.log 2>&1 &
                    echo $! > flask.pid
                '''
            }
        }

        stage('Verify Internal Access') {
            steps {
                sh '''
                    sleep 3
                    echo "[INFO] Testing app on localhost:${PORT}"
                    curl -s http://localhost:${PORT} || echo "[WARN] App not responding yet"
                '''
            }
        }
    }
}
