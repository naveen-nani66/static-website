pipeline {
    agent any
    environment {
        registry = "497339096730.dkr.ecr.us-east-1.amazonaws.com/static-website-repo"
    }
    stages {
        stage('Checkout') {
            steps {
                // Using the git shorthand for simplicity
                git 'https://github.com/naveen-nani66/static-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                // We tag it with the registry path so it's ready to push
                sh "docker build -t $registry:${BUILD_NUMBER} ."
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
                    // Authenticate and push
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $registry"
                    sh "docker push $registry:${BUILD_NUMBER}"
                }
            }
        }

        stage("Cleanup Old Container") {
            steps {
                // This removes the old container if it exists, but won't fail if it doesn't
                sh "docker rm -f website-cont || true"
            }
        }

        stage("Docker Run") {
            steps {
                // Running on 8082 to avoid Jenkins port conflict
                sh "docker run -itd --name website-cont -p 8082:80 $registry:${BUILD_NUMBER}"
            }
        }
    }
    
    post {
        success {
            echo "Deployment Successful! Site available at http://your-server-ip:8082"
        }
        failure {
            echo "Pipeline failed. Check the logs above."
        }
    }
}
