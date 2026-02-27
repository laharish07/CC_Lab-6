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

        }
    stage('Deploy Backend Containers') {
    steps {
        sh '''
        echo "Recreating clean Docker network..."

        # Force remove containers if they exist
        docker rm -f backend1 || true
        docker rm -f backend2 || true
        docker rm -f nginx-lb || true

        # Remove stale network
        docker network rm app-network || true
        docker network create app-network

        echo "Starting backend containers..."

        docker run -d --name backend1 --network app-network backend-app
        docker run -d --name backend2 --network app-network backend-app

        # allow DNS to stabilize
        sleep 5
        '''
    }
}

stage('Deploy NGINX Load Balancer') {
    steps {
        sh '''
        echo "Starting NGINX load balancer..."

        docker rm -f nginx-lb || true

        # 🔥 CRITICAL: wait longer for Docker DNS
        sleep 10

        # 🔥 verify backend DNS before nginx starts
        echo "Checking backend DNS..."
        docker run --rm --network app-network busybox nslookup backend1

        docker run -d \
          --name nginx-lb \
          --network app-network \
          -p 80:80 \
          nginx

        docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

        # full restart to ensure upstream resolution
        docker restart nginx-lb
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
