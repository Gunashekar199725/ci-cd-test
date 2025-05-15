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

        stage('Check Flask Logs') {
            steps {
                sh '''
                    echo "[INFO] Showing last 20 lines of flask.log to check for errors..."
                    tail -n 20 flask.log || echo "flask.log not found"
                '''
            }
        }

        stage('Verify Flask Process') {
            steps {
                script {
                    def retries = 5
                    def delay = 3
                    def pidExists = false
                    for (int i = 0; i < retries; i++) {
                        def pid = sh(script: 'cat flask.pid || echo ""', returnStdout: true).trim()
                        if (pid) {
                            def running = sh(script: "ps -p ${pid} > /dev/null 2>&1 && echo yes || echo no", returnStdout: true).trim()
                            if (running == 'yes') {
                                echo "[INFO] Flask app running with PID ${pid}"
                                pidExists = true
                                break
                            }
                        }
                        echo "[WARN] Flask app process not found, retrying in ${delay} seconds..."
                        sleep(delay)
                    }
                    if (!pidExists) {
                        error("[ERROR] Flask app process not found after retries.")
                    }
                }
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
