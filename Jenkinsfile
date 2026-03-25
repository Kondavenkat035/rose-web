pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('valaxy-docker')
        DOCKER_IMAGE = "kondavenkat035/rose-web:latest"
        AWS_DEFAULT_REGION = "us-east-1"
        // Force the pipeline to use the same config file for all steps
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kondavenkat035/rose-web.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # Connect to EKS and update the config file defined in environment
                aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name new-cluster1

                # Apply manifests
                kubectl apply -f k8s/deployment.yml
                kubectl apply -f k8s/service.yml
                kubectl apply -f k8s/ingress.yml
                '''
            }
        }

        stage('Show App URL') {
            steps {
                sh '''
                echo "Waiting for Ingress LoadBalancer to provision..."
                
                # Retry loop: waits up to 3 minutes for the hostname to appear
                RETRIES=0
                URL=""
                while [ -z "$URL" ] && [ $RETRIES -lt 18 ]; do
                    URL=$(kubectl get ingress k8s-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
                    
                    if [ -z "$URL" ]; then
                        echo "Ingress hostname not ready... waiting 10s ($((RETRIES+1))/18)"
                        sleep 10
                        RETRIES=$((RETRIES+1))
                    fi
                done

                echo "======================================"
                if [ -z "$URL" ]; then
                    echo "ERROR: LoadBalancer timed out. Check AWS Console."
                else
                    echo "Application deployed successfully!"
                    echo ""
                    echo "Access your app at:"
                    echo "Rose Service: http://$URL/rose"
                fi
                echo "======================================"
                '''
            }
        }
    }
}
