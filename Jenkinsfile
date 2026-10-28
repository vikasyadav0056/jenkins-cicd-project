
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
                    docker rm -f ${APP_NAME}-test || true

                    docker run -d \
                      --name ${APP_NAME}-test \
                      -p ${TEST_PORT}:80 \
                      ${IMAGE_NAME}

                    trap 'docker rm -f ${APP_NAME}-test >/dev/null 2>&1 || true' EXIT

                    for i in $(seq 1 15); do
                      if curl -fsS http://localhost:${TEST_PORT}/ \
                        -o /tmp/jenkins-app-test.html; then
                        break
                      fi
                      sleep 2
                    done

                    grep -q 'Welcome to My CI/CD Project!' \
                      /tmp/jenkins-app-test.html

                    echo 'Application test passed!'
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    docker rm -f ${APP_NAME} || true

                    docker run -d \
                      --restart unless-stopped \
                      --name ${APP_NAME} \
                      -p ${APP_PORT}:80 \
                      ${IMAGE_NAME}

                    curl -fsS http://localhost:${APP_PORT}/ \
                      | grep -q 'Welcome to My CI/CD Project!'

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

