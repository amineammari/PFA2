pipeline {
    agent any

    environment {
        IMAGE_NAME = "webapp"
        DOCKER_HUB_USERNAME = "ammariamine"
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

        stage('Build Angular Project') {
            steps {
                sh '''
                    npm install
                    npm run build --prod
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${COMMIT_ID} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $DOCKER_USER/${IMAGE_NAME}:${COMMIT_ID}
                        """
                    }
                }
            }
        }

        stage('Update Deployment YAML') {
            steps {
                script {
                    sh "sed -i 's|richardchesterwood/k8s-fleetman-webapp-angular:release2|${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${COMMIT_ID}|' deployments/webapp.yaml"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f storage/'
                sh 'kubectl apply -f deployments/'
                sh 'kubectl apply -f services/'
                sh 'kubectl get pods -w'
            }
        }
    }
}

