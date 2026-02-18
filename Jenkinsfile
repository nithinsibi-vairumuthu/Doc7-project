pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REPO   = "730335674713.dkr.ecr.ap-south-1.amazonaws.com/enterprise-cicd-app"
    IMAGE_TAG  = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
        sh 'echo "Building branch: ${BRANCH_NAME}"'
      }
    }

    stage('Code Quality Check') {
      steps {
        sh 'echo "Running lint checks (placeholder)"'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          cd app
          docker build -t enterprise-cicd-app:${IMAGE_TAG} .
        '''
      }
    }

   stage('Login to ECR') {
      steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-ecr-creds'
              ]]) {
                    sh '''
                     aws --version
                     aws ecr get-login-password --region ap-south-1 | \
                     docker login --username AWS --password-stdin 730335674713.dkr.ecr.ap-south-1.amazonaws.com
                    '''
        }
      }
    }

   stage('Login to ECR') {
    steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-ecr-creds'
        ]]) {
            sh '''
              aws --version
              aws ecr get-login-password --region ap-south-1 | \
              docker login --username AWS --password-stdin 730335674713.dkr.ecr.ap-south-1.amazonaws.com
            '''
        }
      }

   }
  

    stage('Deploy to DEV') {
      when {
        branch 'dev'
      }
      steps {
        sh '''
          sed -i "s|image: .*|image: 730335674713.dkr.ecr.ap-south-1.amazonaws.com/enterprise-cicd-app:${IMAGE_TAG}|g" k8s/deployment-dev.yaml
          kubectl apply -f k8s/deployment-dev.yaml
          kubectl apply -f k8s/service-dev.yaml
          kubectl apply -f k8s/ingress-dev.yaml
        '''
      }
    }

    stage('Approval for Production') {
      when {
        branch 'main'
      }
      steps {
        input message: "Approve deployment to Production?"
      }
    }

    stage('Deploy to PROD') {
      when {
        branch 'main'
      }
      steps {
        sh '''
          sed -i "s|image: .*|image: 730335674713.dkr.ecr.ap-south-1.amazonaws.com/enterprise-cicd-app:${IMAGE_TAG}|g" k8s/deployment-prod.yaml
          kubectl apply -f k8s/deployment-prod.yaml
          kubectl apply -f k8s/service-prod.yaml
          kubectl apply -f k8s/ingress-prod.yaml
        '''
      }
    }

    stage('Cleanup') {
      steps {
        sh 'docker system prune -f'
      }
    }
  }

  post {
    success {
      echo '✅ Pipeline completed successfully!'
    }
    failure {
      echo '❌ Pipeline failed!'
    }
  }
}
