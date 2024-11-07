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
        
        stage('List Pods in GKE') {
            steps {
                script {
                    // Use kubectl to list the pods in the GKE cluster
                    sh 'kubectl get pods --all-namespaces'
                }
            }
        }
    }
}
