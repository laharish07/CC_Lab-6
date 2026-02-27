pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend image..."

                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Recreating clean Docker network..."

                # stop old containers
                docker rm -f backend1 backend2 nginx-lb || true

                # remove stale network (VERY IMPORTANT)
                docker network rm app-network || true
                docker network create app-network

                echo "Starting backend containers..."

                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # allow Docker DNS to stabilize
                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Starting NGINX load balancer..."

                docker rm -f nginx-lb || true

                # extra wait to avoid DNS race
                sleep 5

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                echo "Copying nginx config..."
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # CRITICAL: restart so nginx re-resolves backend DNS
                docker restart nginx-lb

                echo "NGINX load balancer deployed."
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
