pipeline {
    agent any
    triggers {
        githubPush() 
    }
    environment {
        // Consolidated to match your ECR URI
        REGISTRY   = "497339096730.dkr.ecr.us-east-1.amazonaws.com/static-website"
        REGION     = "us-east-1"
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/naveen-nani66/static-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                // FIXED: Use REGISTRY (not ECR_URL) and --no-cache to ensure title updates
                sh "docker build --no-cache -t ${REGISTRY}:${BUILD_NUMBER} ."
                sh "docker tag ${REGISTRY}:${BUILD_NUMBER} ${REGISTRY}:latest"
            }
        }

        stage("Uploading to ECR") {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                                 credentialsId: 'aws-jenkins', 
                                 accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                                 secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    // Login is still required for the 'docker push' command
                    sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REGISTRY}"
                    sh "docker push ${REGISTRY}:${BUILD_NUMBER}"
                    sh "docker push ${REGISTRY}:latest"
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                script {
                    // Since you use the Secret Helper, we skip the 'create secret' part!
                    
                    // 1. Update the image tag in your deployment file
                    sh "sed -i 's/BUILD_TAG/${BUILD_NUMBER}/g' deploy.yml"

                    // 2. Apply the update
                    sh "kubectl apply -f deploy.yml --validate=false"

                    // 3. Force the restart to ensure the new title is pulled
                    sh "kubectl rollout restart deployment/static-website"
                    sh "kubectl rollout status deployment/static-website"
                }
            }
        }
    }
    
    post {
        success {
            // Clean up workspace to keep EC2 healthy
            sh "docker rmi ${REGISTRY}:${BUILD_NUMBER} || true"
            sh "docker image prune -f"
            echo "Deployment to Minikube Successful!"
        }
    }
}
