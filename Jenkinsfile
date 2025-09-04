pipeline {
  agent any

  environment {
    EUREKA_IMAGE_NAME   = '445198795790.dkr.ecr.eu-west-1.amazonaws.com/eureka-server'
    SERVICEA_IMAGE_NAME = '445198795790.dkr.ecr.eu-west-1.amazonaws.com/servicea'
    SERVICEB_IMAGE_NAME = '445198795790.dkr.ecr.eu-west-1.amazonaws.com/serviceb'
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'main',
            url: 'https://github.com/Jaseel123/eureka-sd.git'
      }
    }

    stage('Login to AWS ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region eu-west-1 | sudo docker login --username AWS --password-stdin 445198795790.dkr.ecr.eu-west-1.amazonaws.com
        '''
      }
    }

    stage('Build and Push Eureka Server Image') {
      steps {
        sh '''
          cd ${WORKSPACE}/eureka-server/
          sudo docker build -t ${EUREKA_IMAGE_NAME}:${BUILD_NUMBER} .
          sudo docker push ${EUREKA_IMAGE_NAME}:${BUILD_NUMBER}
        '''
      }
    }

    stage('Build and Push ServiceA Image') {
      steps {
        sh '''
          cd ${WORKSPACE}/servicea/
          sudo docker build -t ${SERVICEA_IMAGE_NAME}:${BUILD_NUMBER} .
          sudo docker push ${SERVICEA_IMAGE_NAME}:${BUILD_NUMBER}
        '''
      }
    }

    stage('Build and Push ServiceB Image') {
      steps {
        sh '''
          cd ${WORKSPACE}/serviceb/
          sudo docker build -t ${SERVICEB_IMAGE_NAME}:${BUILD_NUMBER} .
          sudo docker push ${SERVICEB_IMAGE_NAME}:${BUILD_NUMBER}
        '''
      }
    }

    stage('Update Deployment YAMLs') {
      steps {
        sh '''
          cd ${WORKSPACE}/
          sed -i "s|image: .*eureka-server.*|image: ${EUREKA_IMAGE_NAME}:${BUILD_NUMBER}|" k8s/eureka-deployment.yaml
          sed -i "s|image: .*servicea.*|image: ${SERVICEA_IMAGE_NAME}:${BUILD_NUMBER}|" k8s/servicea-deployment.yaml
          sed -i "s|image: .*serviceb.*|image: ${SERVICEB_IMAGE_NAME}:${BUILD_NUMBER}|" k8s/serviceb-deployment.yaml
        '''
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          echo "Refreshing ECR pull secret..."
          aws ecr get-login-password --region eu-west-1 | kubectl create secret docker-registry ecr-creds \
            --docker-server=445198795790.dkr.ecr.eu-west-1.amazonaws.com \
            --docker-username=AWS \
            --docker-password="$(aws ecr get-login-password --region eu-west-1)" \
            --dry-run=client -o yaml | kubectl apply -f -

          echo "Applying Kubernetes deployment and services..."
          kubectl apply -f k8s/eureka-deployment.yaml
          kubectl apply -f k8s/servicea-deployment.yaml
          kubectl apply -f k8s/serviceb-deployment.yaml

          kubectl apply -f k8s/eureka-service.yaml
          kubectl apply -f k8s/servicea-service.yaml
          kubectl apply -f k8s/serviceb-service.yaml
        '''
      }
    }
  }
}
