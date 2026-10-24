
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-cicd-demo'
        CONTAINER_NAME = 'jenkins-cicd-app'
    }

    stages {
        stage('Check Docker') {
            steps {
                sh '''
                    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
                    which docker
                    docker --version
                    docker ps
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 127.0.0.1:18081:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:18081/ \
                            -o /dev/null; then
                            echo "Container test passed"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Container test failed"
                    exit 1
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --restart unless-stopped \
                        --name ${CONTAINER_NAME} \
                        -p 8081:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    docker image tag \
                        ${IMAGE_NAME}:${BUILD_NUMBER} \
                        ${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        always {
            sh '''
                export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
                docker rm -f ${CONTAINER_NAME} || true
            '''
        }

        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
