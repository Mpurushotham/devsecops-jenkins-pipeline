pipeline {
    agent any

    environment {
        IMAGE_NAME = "devsecops-flask"
        DOCKER_REGISTRY = "your-dockerhub-username"
        SONARQUBE = "SonarQube"
    }

    tools {
        python "Python3"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/your-repo/devsecops-flask-app.git'
            }
        }

        stage('Static Code Analysis - SonarQube') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Security Scan - Bandit') {
            steps {
                sh 'bandit -r app/ > bandit-report.txt'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push $DOCKER_REGISTRY/$IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                    sh 'terraform validate'
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Ansible Provisioning') {
            steps {
                dir('ansible') {
                    sh 'ansible-playbook playbook.yml'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }

        stage('Post-deployment Security Scan - Trivy') {
            steps {
                sh 'trivy image $DOCKER_REGISTRY/$IMAGE_NAME:latest'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
