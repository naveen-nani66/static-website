pipeline {
    agent any
    triggers {
        githubPush() // This tells the pipeline to wake up on webhook events
    }
    environment {
        registry = "497339096730.dkr.ecr.us-east-1.amazonaws.com/static-website"
        region = "us-east-1"
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/naveen-nani66/static-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_URL}/${REPO_NAME}:latest ."
            }
        }

        stage("Uploading to ECR") {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin $registry"
                    sh "docker push $registry:${BUILD_NUMBER}"
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    script {
                        // 1. Refresh ECR Secret for Kubernetes
                        sh """
                        kubectl delete secret regcred || true
                        kubectl create secret docker-registry regcred \
                            --docker-server=497339096730.dkr.ecr.us-east-1.amazonaws.com \
                            --docker-username=AWS \
                            --docker-password=\$(aws ecr get-login-password --region ${region})
                        """

                        // 2. Replace BUILD_TAG placeholder in your k8s-deployment.yaml
                        sh "sed -i 's/BUILD_TAG/${BUILD_NUMBER}/g' deploy.yml"

                        // 3. Apply to Minikube
                        sh "kubectl apply -f deploy.yml --validate=false"

                        // 4. Wait for healthy rollout
                        sh "kubectl rollout status deployment/static-website"
                    }
                }
            }
        }
    }
    
    post {
        success {
            // Clean up the local image to save that 25GB disk space!
            sh "docker rmi $registry:${BUILD_NUMBER} || true"
            sh "docker image prune -f"
            echo "K8s Deployment Successful!"
        }
    }
}
