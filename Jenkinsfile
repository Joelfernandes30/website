pipeline {
    agent any

    environment {
        GCP_PROJECT = "bustracking-467614"
        GCP_REGION  = "us-central1"
        REPO_NAME   = "edtech-repo"
        IMAGE_NAME  = "edtech-app-frontend"
        GCLOUD_PATH = "/var/jenkins_home/google-cloud-sdk/bin"
    }

    stages {

        stage("Clone GitHub Repo") {
            steps {
                script {
                    echo '📦 Cloning repository...'
                    checkout scmGit(
                        branches: [[name: '*/prod']],
                        userRemoteConfigs: [[
                            credentialsId: 'github-token',
                            url: 'https://github.com/Joelfernandes30/edtech-web.git'
                        ]]
                    )
                }
            }
        }

        stage("Build & Push Frontend Docker Image") {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo "🐳 Building and pushing frontend Docker image..."
                        sh '''
                        export PATH=$PATH:${GCLOUD_PATH}

                        # Authenticate with GCP
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud config set project ${GCP_PROJECT}

                        # Create Artifact Registry repo if missing
                        gcloud artifacts repositories create ${REPO_NAME} \
                            --repository-format=docker \
                            --location=${GCP_REGION} \
                            --description="Docker repository for frontend" || true

                        # Configure Docker for Artifact Registry
                        gcloud auth configure-docker ${GCP_REGION}-docker.pkg.dev --quiet --project=${GCP_PROJECT}

                        # Build frontend Docker image
                        docker build -t ${IMAGE_NAME}:latest ./frontend

                        # Tag and push to Artifact Registry
                        docker tag ${IMAGE_NAME}:latest ${GCP_REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/${IMAGE_NAME}:latest
                        docker push ${GCP_REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/${IMAGE_NAME}:latest
                        '''
                    }
                }
            }
        }

        stage("Deploy Frontend to Cloud Run") {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo '🚀 Deploying frontend to Cloud Run...'
                        sh '''
                        export PATH=$PATH:${GCLOUD_PATH}
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud config set project ${GCP_PROJECT}

                        gcloud run deploy ${IMAGE_NAME} \
                            --image=${GCP_REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/${IMAGE_NAME}:latest \
                            --platform=managed \
                            --region=${GCP_REGION} \
                            --allow-unauthenticated \
                            --port=8080
                        '''
                    }
                }
            }
        }

    }
}
