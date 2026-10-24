
pipeline {
    agent any

    environment {
    PATH+DOCKER = '/usr/local/bin'
    IMAGE_NAME = 'jenkins-cicd-demo'
    CONTAINER_NAME = 'jenkins-cicd-app'
}

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Test') {
            steps {
                sh '''
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
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p 8081:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    docker image tag \
                        ${IMAGE_NAME}:${BUILD_NUMBER} \
                        ${IMAGE_NAME}:latest
                '''
                echo 'Application deployed successfully!'
            }
        }
    }

    post {
        always {
            sh 'docker rm -f ${CONTAINER_NAME} || true'
        }

        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
