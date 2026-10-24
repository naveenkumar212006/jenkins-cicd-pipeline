
pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'naveenkumar21032006'
        IMAGE_NAME = 'naveenkumar21032006/jenkins-cicd-app'
        CONTAINER_NAME = 'jenkins-cicd-app'
        TEST_CONTAINER_NAME = 'jenkins-cicd-test'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                sh '''
                    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
                    docker --version
                    docker info
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"

                    docker build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                        -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Test Application') {
            steps {
                sh '''
                    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"

                    docker rm -f ${TEST_CONTAINER_NAME} || true

                    docker run -d \
                        --name ${TEST_CONTAINER_NAME} \
                        -p 127.0.0.1:18081:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:18081/ \
                            -o /dev/null; then
                            echo "Application test passed!"
                            docker rm -f ${TEST_CONTAINER_NAME}
                            exit 0
                        fi
                        sleep 2
                    done

                    docker logs ${TEST_CONTAINER_NAME} || true
                    docker rm -f ${TEST_CONTAINER_NAME} || true
                    echo "Application test failed!"
                    exit 1
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASSWORD'
                    )
                ]) {
                    sh '''
                        export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"

                        echo "$DH_PASSWORD" | docker login \
                            -u "$DH_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"

                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --restart unless-stopped \
                        --name ${CONTAINER_NAME} \
                        -p 8081:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    echo "Deployment completed!"
                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
            echo 'Docker image pushed and application deployed.'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
