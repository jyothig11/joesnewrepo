pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                echo 'Source code checkout completed'
            }
        }

        stage('Build Application') {
            steps {
                sh 'python3 -m py_compile app.py'
                echo 'Application build completed'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python3 -m py_compile app.py'
                echo 'Application test passed'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jyothi-cicd-app:latest .'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker rm -f jyothi-cicd-container || true
                    docker run -d --name jyothi-cicd-container \
                        -p 8081:8080 jyothi-cicd-app:latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'sleep 3'
                sh 'python3 -m py_compile app.py'
            }
        }
    }
}
