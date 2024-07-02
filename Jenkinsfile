pipeline {
    agent any
    environment {
        AWS_CREDENTIALS_ID = 'aws-credentials' // Jenkins ID for AWS credentials
        ECR_REPO_URI = '058264215547.dkr.ecr.ap-south-1.amazonaws.com/juiceshop' // ECR Repository URI
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/nickhilpatil/juice-shop.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Install Angular CLI Locally') {
            steps {
                sh 'npm install @angular/cli --save-dev'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'cd frontend && npx ng test --watch=false --source-map=true && cd .. && npm run test:server'
            }
        }
        stage('Security Scan with Semgrep') {
            steps {
                sh 'semgrep --config p/ci .'
            }
        }
        stage('Security Scan with Snyk') {
            steps {
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh 'snyk test'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REPO_URI}:latest")
                }
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                script {
                    withCredentials([aws(credentialsId: 'aws-credentials')]) {
                        sh '''
                            aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECR_REPO_URI}
                            docker push ${ECR_REPO_URI}:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    withCredentials([aws(credentialsId: 'aws-credentials')]) {
                        sh '''
                            eb init -p docker juice-shop --region ap-south-1
                            eb create juice-shop-env
                            eb deploy
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}
