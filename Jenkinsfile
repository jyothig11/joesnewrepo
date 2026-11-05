pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '448079094136.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
                echo 'GitHub source code checkout completed'
            }
        }

        stage('Validate Application') {
            steps {
                sh '''
                    python3 -m py_compile user-service/app.py
                    python3 -m py_compile product-service/app.py
                    python3 -m py_compile order-service/app.py
                '''
                echo 'All three services validated'
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    test -f user-service/Dockerfile
                    test -f product-service/Dockerfile
                    test -f order-service/Dockerfile
                    test -f nginx/nginx.conf
                    docker compose config -q
                '''
                echo 'Application checks passed'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build -t ${ECR_REGISTRY}/user-service:${IMAGE_TAG} ./user-service
                    docker build -t ${ECR_REGISTRY}/product-service:${IMAGE_TAG} ./product-service
                    docker build -t ${ECR_REGISTRY}/order-service:${IMAGE_TAG} ./order-service
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} |
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    docker push ${ECR_REGISTRY}/user-service:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/product-service:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/order-service:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    IMAGE_TAG="${IMAGE_TAG}" docker compose \
                        -f "$WORKSPACE/docker-compose.yml" \
                        --project-directory "$WORKSPACE" \
                        -p joesnewrepo pull \
                        user-service product-service order-service

                    IMAGE_TAG="${IMAGE_TAG}" docker compose \
                        -f "$WORKSPACE/docker-compose.yml" \
                        --project-directory "$WORKSPACE" \
                        -p joesnewrepo up -d --no-build
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    for path in health/user health/product health/order; do
                        success=false

                        for i in $(seq 1 15); do
                            if curl -fsS --max-time 3 \
                                http://127.0.0.1:8083/$path; then
                                success=true
                                break
                            fi
                            sleep 2
                        done

                        if [ "$success" != "true" ]; then
                            echo "Deployment verification failed: $path"

                            docker compose \
                                -f "$WORKSPACE/docker-compose.yml" \
                                --project-directory "$WORKSPACE" \
                                -p joesnewrepo logs --tail=100

                            exit 1
                        fi
                    done

                    echo "All three services are healthy"
                '''
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }

        success {
            echo 'Three microservices deployed successfully!'
        }
    }
}
