pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'EyesOnCloud'
        IMAGE_NAME         = 'jenkins-demo-app'
        IMAGE_FULL_NAME    = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}"
        IMAGE_TAG          = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "=== Checkout ==="
                echo "Building image: ${IMAGE_FULL_NAME}:${IMAGE_TAG}"
                echo "Branch : ${GIT_BRANCH}"
                echo "Commit : ${GIT_COMMIT}"
                sh 'ls -la'
            }
        }

        stage('Prepare') {
            steps {
                echo "=== Prepare ==="
                echo "Injecting build version into application..."
                sh "sed -i 's/BUILD_VERSION/Build-${IMAGE_TAG}/g' app/index.html"
                sh 'grep "Version" app/index.html'
                echo "Version injected"
            }
        }

        stage('Build Image') {
            steps {
                echo "=== Build Docker Image ==="
                echo "Building ${IMAGE_FULL_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_FULL_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_FULL_NAME}:${IMAGE_TAG} ${IMAGE_FULL_NAME}:latest"
                echo "Image built and tagged"
                sh "docker images | grep ${IMAGE_NAME}"
            }
        }

        stage('Push Image') {
            steps {
                echo "=== Push to Docker Hub ==="
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    echo "Logged in to Docker Hub"
                    sh "docker push ${IMAGE_FULL_NAME}:${IMAGE_TAG}"
                    echo "Pushed tag: ${IMAGE_TAG}"
                    sh "docker push ${IMAGE_FULL_NAME}:latest"
                    echo "Pushed tag: latest"
                    sh 'docker logout'
                    echo "Logged out from Docker Hub"
                }
            }
        }

        stage('Verify') {
            steps {
                echo "=== Verify Push ==="
                sh "docker pull ${IMAGE_FULL_NAME}:${IMAGE_TAG}"
                sh "docker inspect ${IMAGE_FULL_NAME}:${IMAGE_TAG} | grep -A3 'Labels'"
                echo "Image verified on Docker Hub"
            }
        }

    }

    post {
        always {
            echo "=== Cleanup ==="
            sh "docker rmi ${IMAGE_FULL_NAME}:${IMAGE_TAG} || true"
            sh "docker rmi ${IMAGE_FULL_NAME}:latest || true"
            echo "Local images cleaned up"
        }
        success {
            echo "SUCCESS: ${IMAGE_FULL_NAME}:${IMAGE_TAG} pushed to Docker Hub"
            echo "Pull with: docker pull ${IMAGE_FULL_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "FAILURE: Image build or push failed"
            echo "Check Docker Hub credentials and network connectivity"
        }
    }
}