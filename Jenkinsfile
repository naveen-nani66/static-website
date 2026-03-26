pipeline {
    agent any
    environment{
        registry="497339096730.dkr.ecr.us-east-1.amazonaws.com/static-website-repo"
    }
    stages {
        stage('Checkout') {
            steps {
               checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/naveen-nani66/static-website.git']])
            }
        }
        
        // Building docker image
        stage('Build docker image'){
            steps{
                sh "docker build -t $registry:${BUILD_NUMBER} ."
            }
            post {
                success {
                    echo "hello"
                    // mail to: "naveen.jalla97@gmail.com"
                    // subject: "status of image"
                    // body: "Image successfully built"
                }
            }
        }
                
        // Uploading Docker image into ECR
stage("Uploading to ECR"){
    steps{
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-jenkins',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
            sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $registry"
            sh "docker push $registry:${BUILD_NUMBER}"
        }
    }
}
        
        // post {
        //     success {
        //         mail to: "naveen.jalla97@gmail.com"
        //         subject: "status of image"
        //         body: "Image successfully deployed to ECR"
        //     }
        // }
        
        // stopping previous container and removing
        stage("Stopping previous container"){
            steps{
                sh "docker ps -f name=website-cont -q |xargs --no-run-if-empty docker container stop"
                sh "docker rm website-cont"
                // sh "docker ls -a -f name=website-cont -q | xargs -r docker container rm"
            }
        }
        
        // docker run
        stage("Docker run"){
            steps{
                sh "docker run -itd --name website-cont -p 8082:80 $registry:${BUILD_NUMBER}"
            }
        }
        // post {
        //     success {
        //         mail to: "naveen.jalla97@gmail.com"
        //         subject: "Status of Container"
        //         body: "Image successfully built and container created , you can access with port 8082"
        //     }
        // }
    }
}
pipeline {
    agent any
    environment{
        registry="497339096730.dkr.ecr.us-east-1.amazonaws.com/static-website-repo"
        
    }
    stages {
        stage('Hello') {
            steps {
                git 'https://github.com/naveen-nani66/static-website.git'
            }
        }
        stage ("Build Docker Image"){
            steps{
                sh "docker build -t static-website-image:${BUILD_NUMBER} ."
            }
        }
        stage ("Docker run"){
            steps{
                sh "docker run -itd --name static-website-cont -p 8082:80 static-website-image:${BUILD_NUMBER}"
            }
        }
        stage("AWS Configure"){
            steps{
                withCredentials([[
                    $class:'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
                        sh "aws s3 ls"
                    }
            }
        }
        stage("Docker login"){
            steps{
              sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 497339096730.dkr.ecr.us-east-1.amazonaws.com"
              sh "docker push static-website-image:${BUILD_NUMBER}"
            }
        }
    }
}

