pipeline {
    agent any

    environment {
        IMAGE_NAME = "webapp"
        DOCKER_TLS_VERIFY = "1"
    	DOCKER_HOST = "tcp://192.168.49.2:2376"
    	DOCKER_CERT_PATH = "/home/amin/.minikube/certs"
    	MINIKUBE_ACTIVE_DOCKERD = "minikube"

    }

    stages {
        stage('Preparation') {
            steps {
                checkout scm
                script {
                    COMMIT_ID = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "Commit ID: ${COMMIT_ID}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'eval $(minikube docker-env)'
                    sh "docker build -t ${IMAGE_NAME}:${COMMIT_ID} ."
                }
            }
        }

        stage('Update Deployment YAML') {
            steps {
                script {
                    sh "sed -i 's|richardchesterwood/k8s-fleetman-webapp-angular:release2|${IMAGE_NAME}:${COMMIT_ID}|' k8s-manifests/deployments/webapp.yaml"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s-manifests/storage/'
                sh 'kubectl apply -f k8s-manifests/deployments/'
                sh 'kubectl apply -f k8s-manifests/services/'
                sh 'kubectl get pods'
            }
        }
    }
}

