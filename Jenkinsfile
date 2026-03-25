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
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'k8s'
                ]]) {
                    sh '''
                    export AWS_DEFAULT_REGION=us-east-1

                    # 🔥 VERY IMPORTANT (fix old endpoint issue)
                    rm -rf ~/.kube/config

                    aws eks update-kubeconfig \
                      --region us-east-1 \
                      --name new-cluster1

                    # Debug (optional but useful)
                    kubectl get nodes

                    # Deploy
                    kubectl apply -f k8s/deployment.yml
                    kubectl apply -f k8s/service.yml
                    kubectl apply -f k8s/ingress.yml
                    '''
                }
            }
        }
    }
}
