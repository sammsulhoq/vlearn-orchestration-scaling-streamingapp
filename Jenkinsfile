pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timestamps()
        timeout(time: 45, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
        AWS_PAGER = ''
        PATH = "/usr/local/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Identify AWS Account') {
            steps {
                script {
                    def accountId = sh(
                        script: 'aws sts get-caller-identity --query Account --output text',
                        returnStdout: true
                    ).trim()

                    env.ECR_REGISTRY =
                        "${accountId}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com"

                    env.IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    set -eu

                    docker build \
                      -t "$ECR_REGISTRY/streamingapp/frontend:$IMAGE_TAG" \
                      frontend

                    docker build \
                      -t "$ECR_REGISTRY/streamingapp/auth:$IMAGE_TAG" \
                      backend/authService

                    for service in streaming admin chat; do
                      docker build \
                        -f "backend/${service}Service/Dockerfile" \
                        -t "$ECR_REGISTRY/streamingapp/$service:$IMAGE_TAG" \
                        backend
                    done
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    set -eu

                    aws ecr get-login-password \
                      > "$WORKSPACE/.ecr-password"
                    chmod 600 "$WORKSPACE/.ecr-password"

                    docker login \
                      --username AWS \
                      --password-stdin "$ECR_REGISTRY" \
                      < "$WORKSPACE/.ecr-password"

                    rm -f "$WORKSPACE/.ecr-password"

                    for service in frontend auth streaming admin chat; do
                      docker push \
                        "$ECR_REGISTRY/streamingapp/$service:$IMAGE_TAG"
                    done
                '''
            }
        }
    }

    post {
        always {
            sh '''
                rm -f "$WORKSPACE/.ecr-password"
                if [ -n "${ECR_REGISTRY:-}" ]; then
                  docker logout "$ECR_REGISTRY" || true
                fi
            '''
        }
        success {
            echo "All five images pushed with tag: ${env.IMAGE_TAG}"
        }
    }
}
