pipeline {
    agent any
    stages {
        stage('Pull code from Github') {
            steps {
                git branch: 'main', url: 'https://github.com/soundharajan-S/devops-project.git'
            }
        }
        stage('Code quality check') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'sonar-scanner -Dsonar.projectName=devops-project -Dsonar.projectKey=devops-project'
                }
            }
        }
        stage('docker image build') {
            steps {
                sh 'docker build -t devops-project:latest .'
            }
        }
        stage('Image Scan') {
            steps {
                sh 'trivy image --format table -o image.html devops-project:latest || true'
                sh 'trivy fs --format table -o fs.html . || true'
            }
        }
        stage('Push to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 807598718740.dkr.ecr.ap-south-1.amazonaws.com
                docker tag devops-project:latest 807598718740.dkr.ecr.ap-south-1.amazonaws.com/devops-project:latest
                docker push 807598718740.dkr.ecr.ap-south-1.amazonaws.com/devops-project:latest
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                aws eks update-kubeconfig --region ap-south-1 --name devops-cluster
                kubectl apply -f deployment.yaml
                kubectl rollout restart deployment/devops-project
                kubectl rollout status deployment/devops-project
                '''
            }
        }
    }
}
