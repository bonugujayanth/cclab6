pipeline {
    agent any

    environment {
        NETWORK_NAME = 'lab6-net'
        USE_NETWORK = 'false'
    }

    stages {
        stage('Build Backend Image') {
            steps {
                sh 'docker build -t backend-app backend'
            }
        }

        stage('Create Network') {
            steps {
                script {
                    def status = sh(
                        script: 'docker network inspect lab6-net >/dev/null 2>&1 || docker network create lab6-net',
                        returnStatus: true
                    )

                    if (status == 0) {
                        env.USE_NETWORK = 'true'
                        echo "Using Docker network: ${env.NETWORK_NAME}"
                    } else {
                        env.USE_NETWORK = 'false'
                        echo 'Could not create Docker network. Falling back to --link mode.'
                    }
                }
            }
        }

        stage('Deploy Backends') {
            steps {
                script {
                    if (env.USE_NETWORK == 'true') {
                        sh '''
                            docker rm -f backend1 backend2 || true
                            docker run -d --name backend1 --network lab6-net backend-app
                            docker run -d --name backend2 --network lab6-net backend-app
                            sleep 3
                        '''
                    } else {
                        sh '''
                            docker rm -f backend1 backend2 || true
                            docker run -d --name backend1 backend-app
                            docker run -d --name backend2 backend-app
                            sleep 3
                        '''
                    }
                }
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                script {
                    if (env.USE_NETWORK == 'true') {
                        sh '''
                            docker rm -f nginx-lb || true
                            docker run -d --name nginx-lb --network lab6-net -p 80:80 nginx:latest
                            sleep 2
                            docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                            docker exec nginx-lb nginx -t
                            docker exec nginx-lb nginx -s reload
                        '''
                    } else {
                        sh '''
                            docker rm -f nginx-lb || true
                            docker run -d --name nginx-lb --link backend1:backend1 --link backend2:backend2 -p 80:80 nginx:latest
                            sleep 2
                            docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                            docker exec nginx-lb nginx -t
                            docker exec nginx-lb nginx -s reload
                        '''
                    }
                }
            }
        }

        stage('Verify Running Containers') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully, NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}