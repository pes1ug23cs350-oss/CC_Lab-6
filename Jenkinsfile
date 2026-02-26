pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker network create app-network || true
                docker rm -f backend1 backend2 || true

                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                echo "Waiting for backend to start..."
                sleep 10
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                sleep 3

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer running at http://localhost'
        }
        failure {
            echo 'Pipeline failed. Check console logs.'
        }
    }
}
