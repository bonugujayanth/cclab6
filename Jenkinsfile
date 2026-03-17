pipeline {
    agent any

    stages {
        stage('Build Backend Image') {
            steps {
                sh 'docker build -t backend-app backend'
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    docker rm -f backend1 || true
                    # Use host network; backend listens on 8080 by default
                    docker run -d --name backend1 --network host backend-app
                    sleep 3
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                    docker rm -f nginx-lb || true
                    # Use host network; NGINX listens on 80 by default
                    docker run -d --name nginx-lb --network host nginx:latest
                    sleep 2
                    docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                    docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}