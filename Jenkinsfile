pipeline {
    agent any

    stages {
        stage('Validate Kubernetes') {
            steps {
                bat 'kubectl version --client'
            }
        }

        stage('Deploy Application') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-credentials',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat 'kubectl apply -f deployment.yaml'
                }
            }
        }

        stage('Check Deployment Status') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-credentials',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat 'kubectl rollout status deployment/set14-deployment --timeout=120s'
                }
            }
        }

        stage('Verify Pods') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-credentials',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat 'kubectl get deployment set14-deployment'
                    bat 'kubectl get pods -l app=set14-app'
                    bat 'kubectl get service set14-service'
                }
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed. Collecting Kubernetes diagnostic information.'

            withCredentials([
                file(
                    credentialsId: 'kubeconfig-credentials',
                    variable: 'KUBECONFIG'
                )
            ]) {
                script {
                    bat returnStatus: true, script: 'kubectl get deployments'
                    bat returnStatus: true, script: 'kubectl get pods -l app=set14-app -o wide'
                    bat returnStatus: true, script: 'kubectl describe deployment set14-deployment'
                    bat returnStatus: true, script: 'kubectl get events --sort-by=.metadata.creationTimestamp'
                    bat returnStatus: true, script: 'kubectl logs deployment/set14-deployment --all-pods=true --tail=50'
                }
            }
        }
    }
}