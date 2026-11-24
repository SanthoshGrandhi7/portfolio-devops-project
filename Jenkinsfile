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

        stage('Check Kubernetes and Helm') {
            steps {
                bat 'minikube status -p minikube'
                bat 'kubectl --context=minikube get nodes'
                bat 'helm version'
            }
        }

        stage('Validate Helm Chart') {
            steps {
                bat 'helm lint helm/portfolio'
            }
        }

        stage('Load Image into Minikube') {
            steps {
                bat 'minikube -p minikube image load portfolio-devops:jenkins-%BUILD_NUMBER%'
            }
        }

        stage('Deploy using Helm') {
            steps {
                bat 'helm upgrade --install portfolio helm/portfolio -n portfolio --kube-context minikube --set-string image.tag=jenkins-%BUILD_NUMBER% --wait --timeout 180s'
            }
        }

        stage('Verify Rolling Update') {
            steps {
                bat 'kubectl --context=minikube rollout status deployment/portfolio-deployment -n portfolio --timeout=180s'
                bat 'kubectl --context=minikube get pods -n portfolio'
                bat 'helm list -n portfolio --kube-context minikube'
            }
        }

        stage('Verify Application') {
            steps {
                bat 'kubectl --context=minikube exec -n portfolio deployment/portfolio-deployment -- wget -q -O /dev/null http://portfolio-service/'
            }
        }
    }
}