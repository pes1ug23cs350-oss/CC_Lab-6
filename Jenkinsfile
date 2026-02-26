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
                # Remove old containers if they exist
                docker rm -f backend1 backend2 nginx-lb || true

                # Remove old network if it exists
                docker network rm app-network || true

                # Create fresh network
                docker network create app-network

                # Start backend containers on same network
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Start nginx on same network
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                # Copy updated config
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Give containers time to register in Docker DNS
                sleep 3

                # Reload nginx
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
