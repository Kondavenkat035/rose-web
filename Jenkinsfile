pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('valaxy-docker')
        DOCKER_IMAGE = "kondavenkat035/rose-web:latest"
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
                export AWS_DEFAULT_REGION=us-east-1

                # 🔥 Always refresh kubeconfig (VERY IMPORTANT)
                rm -rf ~/.kube/config

                # Connect to EKS
                aws eks update-kubeconfig \
                  --region us-east-1 \
                  --name new-cluster1

                # Debug (check connection)
                kubectl get nodes

                # Deploy to Kubernetes
                kubectl apply -f k8s/deployment.yml
                kubectl apply -f k8s/service.yml
                kubectl apply -f k8s/ingress.yml
                '''
            }
        }
    }
}
