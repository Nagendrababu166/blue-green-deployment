pipeline {
    agent any

    environment { 
        PROJECT_ID = 'k8s-prod-440909'  
        CLUSTER_NAME = 'prod' 
        CLUSTER_ZONE = 'us-east1-b'
        SERVICE_NAME = 'myapp'
    }
    stages {
        stage('Setup GCP Authentication') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gke-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh '''
                            gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                            gcloud config set project ${PROJECT_ID}
                        '''
                    }
                }
            }
        }
        
        stage('Get GKE Cluster Credentials') {
            steps {
                script {
                    sh '''
                        gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${CLUSTER_ZONE} --project ${PROJECT_ID}
                    '''
                }
            }
        }
        
        stage('Blue Version Deployment') {
            steps {
                script {
                    echo "Deploying Blue (NGINX) version"
                    sh """
                        kubectl apply -f k8s/blue-deployment.yaml
                        kubectl apply -f k8s/blue-service.yaml
                    """
                }
            }
        }
                stage('Green Version Deployment') {
            steps {
                script {
                    echo "Deploying Green (Apache) version"
                    sh """
                        kubectl apply -f k8s/green-deployment.yaml
                        kubectl apply -f k8s/green-service.yaml
                    """
                }
            }
        }
stage('Verify Green Version Deployment') {
    steps {
        script {
            echo "Waiting for Green pods to spin up"
            sh 'sleep 30'
            echo "Verifying Green (Apache) application"
            def greenPods = sh(script: 'kubectl get pods -l app=myapp,version=green -o jsonpath="{.items[*].status.phase}"', returnStdout: true).trim()

            if (!greenPods.contains('Running')) {
                error "Green pods are not running as expected!"
            }
            echo "Green application is running successfully"
        }
    }
}
              stage('Approval to Switch Traffic') {
            steps {
                script {
                    echo "Waiting for manual approval to switch traffic to Green version"
                    input message: 'Approve switch to Green version?', ok: 'Switch Traffic'
                }
            }
        }

        stage('Switch Traffic to Green Version') {
            steps {
                script {
                    echo "Switching traffic to Green (Apache) version"
                    sh '''
                        kubectl patch svc ${SERVICE_NAME} -p '{"spec":{"selector":{"app":"myapp","version":"green"}}}'
                    '''
                }
            }
        }

    }
}
