# Spring Boot Microservices with Eureka Service Discovery and Jenkins CI/CD on Kubernetes

This project demonstrates how to build and deploy containerized Spring Boot microservices using **Eureka for service discovery**, and **Jenkins CI/CD** for automation. The services are deployed on a **Kubernetes cluster** with Docker images pushed to **AWS ECR**.

---

## 📦 Project Structure

```
eureka-sd/
├── eureka-server/           # Eureka Discovery Server (port 8761)
├── servicea/                # Spring Boot service registered with Eureka (port 8081)
├── serviceb/                # Another Spring Boot service (port 8082)
├── k8s/                     # Kubernetes YAML files (Deployments and Services)
└── Jenkinsfile              # Jenkins CI/CD pipeline definition
```

---

## ⚙️ Technologies Used

- Java 17 + Spring Boot
- Spring Cloud Netflix Eureka
- Maven (build tool)
- Docker
- Kubernetes (Kubeadm setup on EC2)
- AWS ECR (Elastic Container Registry)
- Jenkins (CI/CD)
- Calico (CNI plugin)
- GitHub (source code hosting)

---

## 🔧 How It Works

1. **Eureka Server** runs on port `8761` and acts as a service registry.
2. **Service A** registers itself with Eureka and exposes a REST endpoint `/helloWorld`.
3. **Service B** queries Eureka to find Service A's address and calls `/helloWorld` dynamically using `DiscoveryClient`.
4. **Jenkins Pipeline** performs:
   - Git checkout
   - Docker image build and push to ECR
   - Updates image tags in Kubernetes manifests
   - Applies manifests to deploy microservices to the cluster

---

## 🛠️ Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/Jaseel123/eureka-sd.git
cd eureka-sd
```

### 2. Build & Run Locally (for testing)

```bash
cd eureka-server
./mvnw spring-boot:run

cd ../servicea
./mvnw spring-boot:run

cd ../serviceb
./mvnw spring-boot:run
```

### 3. Kubernetes Deployment via Jenkins

- Jenkins Pipeline (`Jenkinsfile`) is triggered on every code push.
- It builds and pushes images to AWS ECR using `BUILD_NUMBER` as the tag.
- Kubernetes YAML files in the `k8s/` directory are updated and applied.

---

## 🔐 AWS ECR Auth & Kubernetes

- Jenkins uses AWS CLI to log in to ECR and push images.
- Kubernetes uses an image pull secret (`ecr-creds`) created dynamically in the pipeline.

---

## 🌐 Access URLs

| Service       | Port   | URL Example                                 |
|---------------|--------|----------------------------------------------|
| Eureka Server | 8761   | `http://<NODE-IP>:30061`                |
| Service A     | 8081   | `http://<NODE-IP>:30081/helloWorld`                         |
| Service B     | 8082   | `http://<NODE-IP>:30082/helloEureka`    |

---
