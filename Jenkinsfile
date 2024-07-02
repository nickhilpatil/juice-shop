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

        stage('Run Tests') {
            steps {
                sh 'npm test'
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
                    withCredentials([string(credentialsId: 'aws-credentials', variable: 'AWS_ACCESS_KEY_ID'),
                                     string(credentialsId: 'aws-credentials', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        docker.withRegistry(ECR_REPO_URI, 'aws-ecr') {
                            dockerImage.push()
                        }
                    }
                }
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    sh 'eb init -p docker juice-shop --region us-west-2'
                    sh 'eb create juice-shop-env'
                    sh 'eb deploy'
                }
            }
        }
    }
}
