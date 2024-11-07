pipeline {
    agent any

    environment {
        GOOGLE_CREDENTIALS = credentials('gke-service-account')  // Replace with your secret ID
        PROJECT_ID = 'k8s-prod-440909'  // Replace with your GCP project ID
        CLUSTER_NAME = 'prod'  // Replace with your GKE cluster name
        CLUSTER_ZONE = 'us-east1-b'  // Replace with your GKE cluster zone
    }
    stages {
        stage('Setup GCP Authentication') {
            steps {
                script {
                    // Authenticate with GCP using the service account credentials
                    withCredentials([file(credentialsId: 'gke-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh '''
                            # Authenticate to GCP using the service account key file
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
                    // Get credentials for the GKE cluster
                    sh '''
                        gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${CLUSTER_ZONE} --project ${PROJECT_ID}
                    '''
                }
            }
        }
        
        stage('Blue Version Deployment') {
            steps {
                script {
                    // Deploy Blue version (NGINX)
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
                    // Deploy Green version (Apache)
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
            // Wait for 30 seconds to give time for the Green pods to spin up
            echo "Waiting for Green pods to spin up"
            sh 'sleep 30'

            // Verify Green deployment is running fine
            echo "Verifying Green (Apache) application"
            def greenPods = sh(script: 'kubectl get pods -l app=myapp,version=green -o jsonpath="{.items[*].status.phase}"', returnStdout: true).trim()

            if (!greenPods.contains('Running')) {
                error "Green pods are not running as expected!"
            }
            echo "Green application is running successfully"
        }
    }
}

    }
}
