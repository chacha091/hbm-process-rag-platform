pipeline {
    agent any

    environment {
        REGISTRY_HOST = 'amdp-registry.skala-ai.com'
        REGISTRY_PROJECT = 'skala26a-ai2'
        BACKEND_IMAGE = 'sk057-backend'
        FRONTEND_IMAGE = 'sk057-frontend'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Code Build') {
            steps {
                sh '''
                    if command -v python3 >/dev/null 2>&1; then
                        python3 --version
                        python3 -m py_compile backend_api.py rag_backend_full.py
                    elif command -v python >/dev/null 2>&1; then
                        python --version
                        python -m py_compile backend_api.py rag_backend_full.py
                    else
                        echo "Python is not installed on this Jenkins agent."
                        exit 1
                    fi
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker --version
                    docker build -t $REGISTRY_HOST/$REGISTRY_PROJECT/$BACKEND_IMAGE:$IMAGE_TAG -f Dockerfile.backend .
                    docker build -t $REGISTRY_HOST/$REGISTRY_PROJECT/$BACKEND_IMAGE:latest -f Dockerfile.backend .

                    docker build -t $REGISTRY_HOST/$REGISTRY_PROJECT/$FRONTEND_IMAGE:$IMAGE_TAG -f Dockerfile.frontend .
                    docker build -t $REGISTRY_HOST/$REGISTRY_PROJECT/$FRONTEND_IMAGE:latest -f Dockerfile.frontend .
                '''
            }
        }

        stage('Push To Harbor') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'harbor-robot-skala26a-ai2',
                    usernameVariable: 'HARBOR_USERNAME',
                    passwordVariable: 'HARBOR_PASSWORD'
                )]) {
                    sh '''
                        echo "$HARBOR_PASSWORD" | docker login $REGISTRY_HOST -u "$HARBOR_USERNAME" --password-stdin

                        docker push $REGISTRY_HOST/$REGISTRY_PROJECT/$BACKEND_IMAGE:$IMAGE_TAG
                        docker push $REGISTRY_HOST/$REGISTRY_PROJECT/$BACKEND_IMAGE:latest

                        docker push $REGISTRY_HOST/$REGISTRY_PROJECT/$FRONTEND_IMAGE:$IMAGE_TAG
                        docker push $REGISTRY_HOST/$REGISTRY_PROJECT/$FRONTEND_IMAGE:latest

                        docker logout $REGISTRY_HOST
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed. Check the Jenkins console output."
        }
        always {
            sh '''
                if command -v docker >/dev/null 2>&1; then
                    docker images | grep -E "sk057-(backend|frontend)" || true
                else
                    echo "Docker is not installed on this Jenkins agent."
                fi
            '''
        }
    }
}
