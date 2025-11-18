pipeline {
    agent any

    environment {
        PROJECT_ID = "applied-pager-476808-j5"
        REGION = "us-central1"
        REPO = "pythonapp"
        // CORRECTED: Define the image with a tag variable for better control
        IMAGE_NAME = "python-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}" // Use Jenkins BUILD_NUMBER for unique tag
        IMAGE = "us-central1-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE_NAME}:${IMAGE_TAG}"

        GKE_CLUSTER = "gke-cluster"
        GKE_ZONE = "us-central1-a"
        GCP_CREDENTIALS = "serviceaccountkey"
        DEPLOYMENT_NAME = "python-gunicorn-app"
        CONTAINER_NAME = "python-gunicorn-app" // Name of the container inside the deployment
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

        stage('Build and Push Docker Image') { // Renamed stage for clarity
            steps {
                // Build and Tag the image using the unique BUILD_NUMBER
                sh "gcloud builds submit --tag ${IMAGE} ."
                // Pushing is typically automatic with 'gcloud builds submit' to GCR/Artifact Registry
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
                // 1. Initial Deployment (If deployment.yaml is present)
                // This ensures the deployment exists on the *first* run.
                sh "if ! kubectl get deployment ${DEPLOYMENT_NAME}; then kubectl apply -f deployment.yaml; fi"

                // 2. Rolling Update
                // CORRECTION: Use the dynamic ${IMAGE} environment variable.
                // CORRECTION: Use the CONTAINER_NAME from the environment block.
                sh """
                    echo "🚀 Rolling update deployment ${DEPLOYMENT_NAME} with new image ${IMAGE}"
                    kubectl set image deployment/${DEPLOYMENT_NAME} \
                    ${CONTAINER_NAME}=${IMAGE}
                """
                // OPTIONAL: Wait for the rollout to complete
                sh "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
            }
        }
    }
}
