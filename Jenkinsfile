def commit_id
pipeline {
    agent any 


    stages {
        stage('Preparation') {
            steps {
                checkout scm
                sh"git rev-parse --short HEAD > .git/commit-id"
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stage('Image Build'){
            steps {
                echo 'Building.....'
                sh 'scp -r -i $(/home/amin/.minikube/machines/minikube/id_rsa) ./* docker@$(192.168.49.2):~/'
                sh "minikube ssh 'docker build -t webapp:${commit_id} ./'"
                echo 'build complete'
            }
        }
        stage('Deploy'){
            steps {
                echo 'Deploying to Kubernetes'
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-webapp-angular:release2|webapp:${commit_id}|' ./manifests.workloads.yaml"
                sh 'kubectl get all'
                sh 'kubectl apply -f ./manifests/ '
                sh 'kubectl get all'
                echo 'deployment complete'
            }
        }
    }
}
