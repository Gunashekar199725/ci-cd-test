pipeline {
    agent any

    tools {
      git 'Git' // Make sure this matches the name in Global Tool Configuration
   }

    stages {
        stage('Clone Repo') {
            steps {
                echo 'Cloning repo...'
            }
        }

        stage('Set Up Python') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install flask
                '''
            }
        }

        stage('Run Flask App') {
            steps {
                sh '''                    
                    . venv/bin/activate
                    nohup python3 app.py > app.log 2>&1 &
                '''
            }
        }
    }
}
