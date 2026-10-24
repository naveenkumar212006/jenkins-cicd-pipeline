
pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'

                sh '''
                    test -f app/index.html
                    mkdir -p public
                    cp app/index.html public/index.html
                '''

                echo 'Build completed successfully!'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing the application...'

                sh '''
                    test -s public/index.html
                    grep -q "Jenkins CI/CD Pipeline" public/index.html
                    grep -q "Build, Test and Deploy" public/index.html
                '''

                echo 'All tests passed!'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the application...'

                sh '''
                    mkdir -p deployed
                    cp public/index.html deployed/index.html
                    test -s deployed/index.html
                '''

                echo 'Application deployed to the workspace!'
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
