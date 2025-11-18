pipeline {
    agent any

    environment {
        PROJECT_ID = "applied-pager-476808-j5"
        REGION = "us-central1"
        REPO = "pythonapp"
        IMAGE = "us-central1-docker.pkg.dev/${PROJECT_ID}/${REPO}/python-app:latest"
        GKE_CLUSTER = "gke-cluster"
        GKE_ZONE = "us-central1-a"
        GCP_CREDENTIALS = "serviceaccountkey"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/TCRDINSEH/DevOpsProjectWithGCP.git'
            }   
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: "${GCP_CREDENTIALS}", variable: 'GCP_KEYFILE')]) {
                    sh """
                       echo "🔐 Authenticating to GCP..."
                       gcloud auth activate-service-account --key-file=$GCP_KEYFILE
                       gcloud config set project ${PROJECT_ID}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "gcloud builds submit --tag ${IMAGE} ."
            }
        }

        stage('Get GKE Credentials') {
            steps {
                sh """
                    gcloud container clusters get-credentials ${GKE_CLUSTER} \
                    --zone ${GKE_ZONE} --project ${PROJECT_ID}
                """
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh """
                    kubectl set image deployment/python-gunicorn-app \
                    python-gunicorn-app=${IMAGE} --record
                """
            }
        }
    }
}
