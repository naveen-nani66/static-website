pipeline {
    agent any
    triggers {
        githubPush() 
    }
    environment {
        REGISTRY = "497339096730.dkr.ecr.us-east-1.amazonaws.com/static-website"
        REGION   = "us-east-1"
    }
    stages {
        stage('Disk Health Check') {
            steps {
                script {
                    // Check if disk usage is above 90% before starting
                    def usage = sh(script: "df / --output=pcent | tail -1 | tr -dc '0-9'", returnStdout: true).trim().toInteger()
                    if (usage > 90) {
                        error "Build Aborted: Disk is at ${usage}%. Clean space to protect Postgres/Grafana!"
                    }
                    echo "Disk is healthy at ${usage}%. Proceeding..."
                }
            }
        }

        stage('Checkout') {
            steps {
                git 'https://github.com/naveen-nani66/static-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Using --pull ensures we have latest base without bloating local cache
                sh "docker build --pull --no-cache -t ${REGISTRY}:${BUILD_NUMBER} ."
                sh "docker tag ${REGISTRY}:${BUILD_NUMBER} ${REGISTRY}:latest"
            }
        }

        stage("Uploading to ECR") {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                                  credentialsId: 'aws-jenkins', 
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REGISTRY}"
                    sh "docker push ${REGISTRY}:${BUILD_NUMBER}"
                    sh "docker push ${REGISTRY}:latest"
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                script {
                    sh "sed -i 's/BUILD_TAG/${BUILD_NUMBER}/g' deploy.yml"
                    sh "kubectl apply -f deploy.yml --validate=false"
                    sh "kubectl rollout restart deployment/static-website"
                    sh "kubectl rollout status deployment/static-website"
                }
            }
        }
    }
    
    post {
        always {
            // CRITICAL: Clean up the image we just built to save space
            sh "docker rmi ${REGISTRY}:${BUILD_NUMBER} || true"
            // Clean up dangling layers (the 83% disk usage protection)
            sh "docker image prune -f"
        }
        success {
            echo "Deployment Successful! Monitoring remains healthy."
        }
    }
}
