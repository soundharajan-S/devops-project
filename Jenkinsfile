pipeline {
 agent any
 stages {
  stage('Git Checkout') { 
   steps { 
    git branch: 'main', url: 'https://github.com/soundharajan-S/devops-project.git' 
   } 
  }
  stage('SonarQube Scan') { 
   steps { 
    withSonarQubeEnv('sonar') { 
      sh 'sonar-scanner -Dsonar.projectName=devops-project -Dsonar.projectKey=devops-project' 
    } 
   } 
  }
  stage('Trivy FS Scan') { steps { sh 'trivy fs --format table -o fs.html .' } }
  stage('Docker Build') { steps { sh 'docker build -t devops-project:latest .' } }
  stage('Trivy Image Scan') { steps { sh 'trivy image --format table -o image.html devops-project:latest' } }
  stage('Push to ECR') { 
   steps { 
    sh '''
      aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 807598718740.dkr.ecr.ap-south-1.amazonaws.com
      docker tag devops-project:latest 807598718740.dkr.ecr.ap-south-1.amazonaws.com/devops-project:latest
      docker push 807598718740.dkr.ecr.ap-south-1.amazonaws.com/devops-project:latest
    ''' 
   } 
  }
  stage('Deploy to EKS') { 
   steps { 
    sh '''
      aws eks update-kubeconfig --region ap-south-1 --name devops-cluster
      kubectl apply -f deployment.yaml
      kubectl rollout status deployment/devops-project
      kubectl get svc devops-service
    ''' 
   } 
  }
 }
}
