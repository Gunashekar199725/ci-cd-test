pipeline {
    agent any

    tools {
        git 'Default' // Optional: Match with your Git tool name in Global Tool Configuration
    }

    environment {
        VENV = "venv"
        PORT = "5000"
    }

    stages {
        stage('Setup Python & Flask') {
            steps {
                sh '''
                    set -e
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
                        kill $(cat flask.pid) || true
                        sleep 2
                        if kill -0 $(cat flask.pid) 2>/dev/null; then
                            kill -9 $(cat flask.pid)
                        fi
                        rm flask.pid
                    fi
                '''
            }
        }

        stage('Run Flask App in Background') {
            steps {
                sh '''
                    set -e
                    echo "[INFO] Starting Flask app on 0.0.0.0:${PORT}"
                    nohup ${VENV}/bin/python app.py > flask.log 2>&1 &
                    echo $! > flask.pid
                '''
            }
        }

        stage('Verify Internal Access') {
            steps {
                sh '''
                    set +e
                    sleep 3
                    echo "[INFO] Testing app on localhost:${PORT}"
                    curl -s http://localhost:${PORT} || echo "[WARN] App not responding yet"
                '''
            }
        }
    }

    post {
        always {
            echo "[INFO] Archiving Flask log file..."
            archiveArtifacts artifacts: 'flask.log', allowEmptyArchive: true
        }
    }
}
