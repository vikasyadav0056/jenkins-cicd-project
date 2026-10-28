
pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        APP_NAME = 'my-web-app'
        IMAGE_NAME = 'my-web-app:latest'
        TEST_PORT = '8082'
        APP_PORT = '8081'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing the application...'
                sh '''
                    docker rm -f my-web-app-test || true

                    docker run -d \
                      --name my-web-app-test \
                      -p 8082:80 \
                      my-web-app:latest

                    trap 'docker rm -f my-web-app-test >/dev/null 2>&1 || true' EXIT

                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:8082/ \
                            -o /tmp/jenkins-app-test.html; then
                            break
                        fi
                        sleep 2
                    done

                    test -s /tmp/jenkins-app-test.html
                    grep -qi '<html' /tmp/jenkins-app-test.html

                    echo 'Application test passed!'
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    docker rm -f my-web-app || true

                    docker run -d \
                      --restart unless-stopped \
                      --name my-web-app \
                      -p 8081:80 \
                      my-web-app:latest

                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:8081/ \
                            -o /tmp/jenkins-app-deploy.html; then
                            break
                        fi
                        sleep 2
                    done

                    test -s /tmp/jenkins-app-deploy.html
                    grep -qi '<html' /tmp/jenkins-app-deploy.html

                    echo 'Deployment successful!'
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}

