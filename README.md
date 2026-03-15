# Blue-Green Deployment - Banking Application

A Spring Boot banking web application with Blue-Green deployment strategy on AWS EKS using Jenkins CI/CD pipeline.

## Overview

This project demonstrates a zero-downtime deployment approach using the Blue-Green deployment pattern. It enables seamless traffic switching between two identical production environments (Blue and Green), allowing for safe rollbacks and minimal risk during releases.

## Tech Stack

- **Application**: Spring Boot 3.3.3 (Java 17)
- **Database**: MySQL 8
- **Container Runtime**: Docker (Eclipse Temurin JDK 17 Alpine)
- **Orchestration**: Kubernetes (AWS EKS)
- **CI/CD**: Jenkins
- **Infrastructure**: Terraform
- **Security Scanning**: SonarQube, Trivy

## Project Structure

```
.
├── Cluster/                    # Terraform infrastructure files
│   ├── main.tf                 # EKS cluster configuration
│   ├── variables.tf            # Terraform variables
│   ├── output.tf               # Terraform outputs
│   └── monitor/                # Monitoring configuration
├── src/                        # Spring Boot application source
├── Dockerfile                  # Container image definition
├── Jenkinsfile                 # CI/CD pipeline definition
├── app-deployment-blue.yml     # Blue environment deployment
├── app-deployment-green.yml    # Green environment deployment
├── bankapp-service.yml         # Kubernetes LoadBalancer service
├── mysql-ds.yml                # MySQL deployment and service
├── pom.xml                     # Maven build configuration
└── Setup-RBAC.md               # RBAC setup instructions
```

## Prerequisites

- AWS Account with EKS access
- Jenkins server with required plugins:
  - Docker Pipeline
  - Kubernetes CLI
  - SonarQube Scanner
- Docker Hub account
- kubectl configured for your EKS cluster
- Trivy installed on Jenkins agent
- SonarQube server

## Deployment

### 1. Infrastructure Setup

Deploy the EKS cluster using Terraform:

```bash
cd Cluster
terraform init
terraform plan
terraform apply
```

### 2. Configure Jenkins

Set up the following credentials in Jenkins:
- `git-cred` - Git repository credentials
- `docker-cred` - Docker Hub credentials
- `k8-token` - Kubernetes service account token

### 3. Run the Pipeline

The Jenkins pipeline accepts the following parameters:

| Parameter | Options | Description |
|-----------|---------|-------------|
| `DEPLOY_ENV` | blue, green | Target deployment environment |
| `DOCKER_TAG` | blue, green | Docker image tag |
| `SWITCH_TRAFFIC` | true, false | Switch traffic to the deployed environment |

### 4. Traffic Switching

To switch traffic between environments:
1. Deploy to the inactive environment (e.g., green)
2. Verify the new deployment is healthy
3. Run the pipeline with `SWITCH_TRAFFIC=true` to redirect traffic

## Pipeline Stages

1. **Git Checkout** - Clone the repository
2. **SonarQube Analysis** - Static code analysis
3. **Trivy FS Scan** - Filesystem vulnerability scan
4. **Docker Build** - Build container image
5. **Trivy Image Scan** - Container image vulnerability scan
6. **Docker Push** - Push image to Docker Hub
7. **Deploy MySQL** - Deploy MySQL database
8. **Deploy Service** - Create/update LoadBalancer service
9. **Deploy to Kubernetes** - Deploy Blue or Green environment
10. **Switch Traffic** - Update service selector (optional)
11. **Verify Deployment** - Confirm deployment status

## Local Development

### Build the Application

```bash
./mvnw clean package
```

### Run with Docker

```bash
docker build -t bankapp:local .
docker run -p 8080:8080 bankapp:local
```

## Configuration

The application connects to MySQL using these environment variables:

| Variable | Description |
|----------|-------------|
| `SPRING_DATASOURCE_URL` | MySQL JDBC connection URL |
| `SPRING_DATASOURCE_USERNAME` | Database username |
| `SPRING_DATASOURCE_PASSWORD` | Database password |

## License

This project is provided for educational and demonstration purposes.
