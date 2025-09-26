pipeline {
    agent any
    environment {
        SONARQUBE_SERVER = 'SonarQubeServer'  // Name configured in Jenkins -> Manage Jenkins -> Configure System -> SonarQube Servers
        ECR_REPO = '813270451126.dkr.ecr.ap-south-1.amazonaws.com/myrepo'
        AWS_REGION = 'ap-south-1'
        IMAGE_NAME = 'newimage'
        PATH = "/opt/sonar-scanner/bin:$PATH"
    }
    stages {
        stage('Pull Code From GitHub') {
            steps {
                git 'https://github.com/Dharshinivk30/project.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'sonar-scanner'
                    }
                }
            }
        stage('Build the Docker image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
                }
            }
        stage('Image Scan') {
            steps {
                sh 'trivy image ${IMAGE_NAME}:latest > report.txt'
                }
            }
        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                        docker tag $IMAGE_NAME:latest $ECR_REPO:$BUILD_NUMBER
                        docker push $ECR_REPO:$BUILD_NUMBER
                    '''
                    }
                }
            }
        stage('Deploy') {
            steps {
                sh '''
                    sed -i "s|_IMAGE_|$ECR_REPO:$BUILD_NUMBER|g" pod.yaml
                    kubectl apply -f pod.yaml    
                '''
                }
            }
        }
    post {
        success {
            mail to: 'dharshinivk30@gmail.com',
                 subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: "Good news! Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' completed successfully. \nCheck it here: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'dharshinivk30@gmail.com',
                 subject: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: "Oops! Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' has failed. \nCheck logs here: ${env.BUILD_URL}"
        }
    }
}