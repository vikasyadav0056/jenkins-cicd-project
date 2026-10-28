
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

