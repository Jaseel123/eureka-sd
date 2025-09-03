# eureka-sd
# eureka-sd
apiVersion: apps/v1
kind: Deployment
metadata:
  name: servicea
spec:
  replicas: 1
  selector:
    matchLabels:
      app: servicea
  template:
    metadata:
      labels:
        app: servicea
    spec:
      containers:
        - name: servicea
          image: 445198795790.dkr.ecr.eu-west-1.amazonaws.com/servicea-server:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8761
      imagePullSecrets:
        - name: ecr-creds



kubectl create secret docker-registry ecr-creds \
  --docker-server=445198795790.dkr.ecr.eu-west-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region eu-west-1) \
  --docker-email=none


apiVersion: v1
kind: Service
metadata:
  name: servicea
spec:
  type: NodePort
  selector:
    app: servicea
  ports:
    - port: 8081
      targetPort: 8081 
      nodePort: 30081  


      jenkins - admin - admin
      server devops devops

      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
