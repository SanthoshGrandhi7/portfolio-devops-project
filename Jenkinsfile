pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    environment {
        MINIKUBE_HOME = 'C:/Users/santh'
        KUBECONFIG = 'C:/Users/santh/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t portfolio-devops:jenkins-%BUILD_NUMBER% .'
            }
        }

        stage('Verify Docker Image') {
            steps {
                bat 'docker image inspect portfolio-devops:jenkins-%BUILD_NUMBER% >NUL'
            }
        }

        stage('Check Kubernetes') {
            steps {
                bat 'minikube status -p minikube'
                bat 'kubectl --context=minikube get nodes'
            }
        }

        stage('Load Image into Minikube') {
            steps {
                bat 'minikube -p minikube image load portfolio-devops:jenkins-%BUILD_NUMBER%'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl --context=minikube set image --local -f k8s/deployment.yaml portfolio=portfolio-devops:jenkins-%BUILD_NUMBER% -o yaml > deployment-rendered.yaml'

                bat 'kubectl --context=minikube apply -f deployment-rendered.yaml'

                bat 'kubectl --context=minikube apply -f k8s/service.yaml'
            }
        }

        stage('Verify Rolling Update') {
            steps {
                bat 'kubectl --context=minikube rollout status deployment/portfolio-deployment -n portfolio --timeout=180s'

                bat 'kubectl --context=minikube get pods -n portfolio'
            }
        }

        stage('Verify Application') {
            steps {
                bat 'kubectl --context=minikube exec -n portfolio deployment/portfolio-deployment -- wget -q -O /dev/null http://portfolio-service/'
            }
        }
    }
}