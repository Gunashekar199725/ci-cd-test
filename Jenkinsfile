pipeline {
    agent any

    environment {
        VENV = "venv"
        PORT = "5000"
        WORKSPACE = "${env.WORKSPACE}"
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
                    echo "[INFO] Starting Flask app on 0.0.0.0:${PORT} using setsid"
                    # Use setsid to fully detach process
                    setsid nohup ${VENV}/bin/python3 app.py > flask.log 2>&1 < /dev/null &
                    echo $! > flask.pid
                    sleep 3
                    echo "[INFO] Running processes:"
                    ps -ef | grep app.py | grep -v grep || echo "[WARN] Flask app process not found"
                '''
            }
        }

        stage('Verify Internal Access') {
            steps {
                sh '''
                    echo "[INFO] Testing app on localhost:${PORT}"
                    sleep 3
                    if curl -s http://localhost:${PORT}; then
                        echo "[SUCCESS] App responded on port ${PORT}"
                    else
                        echo "[ERROR] App did not respond. Dumping flask.log:"
                        cat flask.log
                        exit 1
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "[INFO] Build finished."
            sh '''
                echo "[INFO] Flask app process status after build:"
                if [ -f flask.pid ]; then
                    ps -p $(cat flask.pid) || echo "[WARN] Flask app process not running"
                else
                    echo "[WARN] No flask.pid file found"
                fi
            '''
        }
    }
}
