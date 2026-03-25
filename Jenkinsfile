pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('valaxy-docker')
        DOCKER_IMAGE = "kondavenkat035/rose-web:latest"
        AWS_DEFAULT_REGION = "us-east-1"
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
                aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name new-cluster1

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
                    echo "Rose Service: http://$URL/rose"
                fi
                echo "======================================"
                '''
            }
        }
    }

    post {
        success {
            emailext (
                subject: "✅ SUCCESS: EKS Deployment #${BUILD_NUMBER}",
                body: """
                <h2>Pipeline Success</h2>
                <p><b>Job:</b> ${JOB_NAME}</p>
                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                <p>Status: SUCCESS ✅</p>
                <p>Your application has been deployed successfully to Kubernetes (EKS).</p>
                """,
                to: "kondavenkat035@gmail.com",
                from: "kondavenkat035@gmail.com"
            )
        }

        failure {
            emailext (
                subject: "❌ FAILED: EKS Deployment #${BUILD_NUMBER}",
                body: """
                <h2>Pipeline Failed</h2>
                <p><b>Job:</b> ${JOB_NAME}</p>
                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                <p>Status: FAILURE ❌</p>
                <p>Please check Jenkins logs and fix the issue.</p>
                """,
                to: "kondavenkat035@gmail.com",
                from: "kondavenkat035@gmail.com"
            )
        }
    }
}
