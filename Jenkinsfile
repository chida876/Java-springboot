pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        EKS_CLUSTER = 'eks-cluster-name'
        DEV_NAMESPACE = 'dev'
        QA_NAMESPACE = 'qa'
        PROD_NAMESPACE = 'production'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-username/your-repo.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("your-docker-image-name")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS) {
                        docker.image("your-docker-image-name").push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'trivy your-docker-image-name:${env.BUILD_NUMBER}'
            }
        }
        
        stage('Deploy to EKS') {
            parallel {
                stage('Deploy to Dev') {
                    when {
                        branch 'dev'
                    }
                    steps {
                        sh 'kubectl apply -f dev-deployment.yaml --namespace=${DEV_NAMESPACE}'
                    }
                }
                stage('Deploy to QA') {
                    when {
                        branch 'qa'
                    }
                    steps {
                        sh 'kubectl apply -f qa-deployment.yaml --namespace=${QA_NAMESPACE}'
                    }
                }
                stage('Deploy to Production') {
                    when {
                        branch 'main'
                    }
                    steps {
                        sh 'kubectl apply -f prod-deployment.yaml --namespace=${PROD_NAMESPACE}'
                    }
                }
            }
        }
    }
    
    post {
        success {
            script {
                def coverage = sh(script: 'mvn jacoco:report | grep "Total instruction coverage"', returnStdout: true).trim()
                if (coverage.contains('80%')) {
                    emailext body: "Code coverage reached 80%: ${coverage}",
                        subject: "Code Coverage Alert",
                        to: "your-email@example.com"
                }
            }
        }
    }
}

