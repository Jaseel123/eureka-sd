pipeline {
    agent any

    environment {
        EUREKA_IMAGE_NAME   = '445198795790.dkr.ecr.eu-west-1.amazonaws.com/eureka-server'
        SERVICEA_IMAGE_NAME = '445198795790.dkr.ecr.eu-west-1.amazonaws.com/servicea'
        SERVICEB_IMAGE_NAME = '445198795790.dkr.ecr.eu-west-1.amazonaws.com/serviceb'
        AWS_REGION          = 'eu-west-1'
        ECR_REGISTRY        = '445198795790.dkr.ecr.eu-west-1.amazonaws.com'
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
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Build and Push Docker Images') {
            parallel {
                stage('Eureka Server') {
                    steps {
                        sh """
                        cd ${WORKSPACE}/eureka-server
                        docker build -t ${EUREKA_IMAGE_NAME}:${BUILD_NUMBER} .
                        docker push ${EUREKA_IMAGE_NAME}:${BUILD_NUMBER}
                        """
                    }
                }

                stage('Service A') {
                    steps {
                        sh """
                        cd ${WORKSPACE}/servicea
                        docker build -t ${SERVICEA_IMAGE_NAME}:${BUILD_NUMBER} .
                        docker push ${SERVICEA_IMAGE_NAME}:${BUILD_NUMBER}
                        """
                    }
                }

                stage('Service B') {
                    steps {
                        sh """
                        cd ${WORKSPACE}/serviceb
                        docker build -t ${SERVICEB_IMAGE_NAME}:${BUILD_NUMBER} .
                        docker push ${SERVICEB_IMAGE_NAME}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Update Deployment YAMLs') {
            steps {
                sh """
                sed -i "s|image: .*eureka-server.*|image: ${EUREKA_IMAGE_NAME}:${BUILD_NUMBER}|" k8s/eureka-deployment.yaml
                sed -i "s|image: .*servicea.*|image: ${SERVICEA_IMAGE_NAME}:${BUILD_NUMBER}|" k8s/servicea-deployment.yaml
                sed -i "s|image: .*serviceb.*|image: ${SERVICEB_IMAGE_NAME}:${BUILD_NUMBER}|" k8s/serviceb-deployment.yaml
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                echo "Refreshing ECR pull secret..."
                aws ecr get-login-password --region ${AWS_REGION} | kubectl create secret docker-registry ecr-creds \
                    --docker-server=${ECR_REGISTRY} \
                    --docker-username=AWS \
                    --docker-password="$(aws ecr get-login-password --region ${AWS_REGION})" \
                    --dry-run=client -o yaml | kubectl apply -f -

                echo "Applying Kubernetes deployments..."
                kubectl apply -f k8s/eureka-deployment.yaml
                kubectl apply -f k8s/servicea-deployment.yaml
                kubectl apply -f k8s/serviceb-deployment.yaml

                echo "Applying Kubernetes services..."
                kubectl apply -f k8s/eureka-service.yaml
                kubectl apply -f k8s/servicea-service.yaml
                kubectl apply -f k8s/serviceb-service.yaml
                """
            }
        }
    }
}
