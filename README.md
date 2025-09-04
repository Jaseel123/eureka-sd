# Spring Boot Microservices with Eureka and Jenkins CI/CD on Kubernetes

## 1. Objective

This project demonstrates how to:

- Build containerized Java Spring Boot microservices.
- Enable service discovery using Eureka.
- Deploy the services to a Kubernetes cluster using YAML manifests.
- Automate build and deployment using a Jenkins CI/CD pipeline.
- Push container images to AWS ECR.

---

## 2. Infrastructure Overview

| Component        | Purpose                        | Configuration               |
|------------------|--------------------------------|------------------------------|
| EC2-1 (Jenkins)   | Jenkins server for CI/CD       | Ubuntu 22.04, Docker, Jenkins |
| EC2-2 (K8s Master)| Kubeadm-based K8s cluster      | Ubuntu 22.04    |

---

## 3. Jenkins Setup

- Jenkins is installed on EC2-1.
- Docker installed and configured (`jenkins` user added to the `docker` group).
- IAM credentials provided for AWS ECR login.
- CI/CD pipeline is defined in `Jenkinsfile`.

---

## 4. Kubernetes Setup

- Kubernetes cluster bootstrapped using `kubeadm`.
- Calico used as CNI plugin.
- kubeconfig copied to Jenkins server for deployment access.
- Docker registry secret (`ecr-creds`) created and refreshed during each deployment.

---

## 5. Project Structure

```bash
eureka-sd/
├── eureka-server/
│   └── Dockerfile
├── servicea/
│   └── Dockerfile
├── serviceb/
│   └── Dockerfile
├── k8s/
│   ├── eureka-deployment.yaml
│   ├── servicea-deployment.yaml
│   ├── serviceb-deployment.yaml
│   ├── eureka-service.yaml
│   ├── servicea-service.yaml
│   └── serviceb-service.yaml
├── Jenkinsfile
└── README.md
```

---

## 6.  Spring Boot + Eureka Setup

### Eureka Server
- Bootstrapped using Spring Initializr with `Eureka Server` dependency.
- `@EnableEurekaServer` enabled in main class.
- application.yml:
  ```yaml
  spring:
    application:
      name: eureka-server
  server:
    port: 8761
  eureka:
    client:
      register-with-eureka: false
      fetch-registry: false
  ```

### ServiceA and ServiceB
- Bootstrapped with `Spring Web` and `Eureka Discovery Client`.
- application.yml for both:
  ```yaml
  spring:
    application:
      name: servicea # or serviceb
  server:
    port: 8081 # or 8082
  eureka:
    client:
      service-url:
        defaultZone: http://eureka:8761/eureka/
    instance:
      prefer-ip-address: true
  ```
- ServiceB calls ServiceA using `DiscoveryClient`.

---

## 7. ⚙️ CI/CD Pipeline (Jenkins)

### Jenkinsfile Stages:
1. **Checkout Code** from GitHub.
2. **ECR Login** using AWS credentials.
3. **Build and Push Docker Images** for each service (tagged with `$BUILD_NUMBER`).
4. **Update Deployment YAMLs** with new image tags using `sed`.
5. **Refresh ECR Secret** in Kubernetes cluster (`kubectl create secret ...`).
6. **Apply Kubernetes Manifests** using `kubectl apply`.

---

## 8. Verifying the Setup

- Access Eureka dashboard via NodePort: `http://<NodeIP>:<NodePort>`
- Use `kubectl get pods`, `logs`, `svc` to verify pod status and services.
- Call `curl http://<NodeIP>:<ServiceB_NodePort>/helloEureka` to test end-to-end discovery.
  - This should route to ServiceA and return: `Hello world from Service A!`

---

## 9. Dockerfile Explained (Multi-Stage Build)

Each microservice uses the following Dockerfile:

```Dockerfile
# ---- Build stage ----
FROM maven:3.9.8-eclipse-temurin-21 AS build
WORKDIR /build
COPY pom.xml .
COPY src ./src
RUN mvn -q -DskipTests package

# ---- Run stage ----
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /build/target/*.jar app.jar
EXPOSE 8082
ENV JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75.0"
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

### Explanation:
- **Build Stage**:
  - Uses Maven to compile and package the application JAR.
- **Run Stage**:
  - Uses a lightweight Java runtime.
  - Only the JAR is copied from the build image.
  - Sets memory limits using `JAVA_TOOL_OPTIONS`.

###  Location:
- `eureka-server/Dockerfile`
- `servicea/Dockerfile`
- `serviceb/Dockerfile`

---

## 10. Repo & Resources

- GitHub Repo: [https://github.com/Jaseel123/eureka-sd](https://github.com/Jaseel123/eureka-sd)
- Based on Spring Guide: [Service Registration and Discovery](https://spring.io/guides/gs/service-registration-and-discovery/)

---

## Summary

This project successfully demonstrates:

- Microservice architecture using Spring Boot
- Service discovery with Eureka
- Containerized deployment via Docker & Kubernetes
- CI/CD automation with Jenkins
- Secure image hosting on AWS ECR

---
