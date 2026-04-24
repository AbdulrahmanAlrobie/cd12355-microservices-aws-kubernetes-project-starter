# Coworking Space Analytics Service

## Overview
This project deploys a Flask-based analytics API for the Coworking Space Service onto AWS EKS using a containerized microservices architecture. The API provides business analysts with usage reports from a PostgreSQL database running inside the Kubernetes cluster.

## Technologies Used
- **Docker** - containerize the Python analytics application
- **AWS ECR** - store and version Docker images
- **AWS EKS** - run the application on Kubernetes
- **AWS CodeBuild** - automate the build and push pipeline
- **AWS CloudWatch** - monitor application logs via Container Insights
- **PostgreSQL** - database deployed as a Kubernetes service with persistent EBS storage

## Dependencies

### Local Environment
1. Python 3.11+ - run the analytics application locally
2. Docker - build and run Docker images
3. `kubectl` - interact with the Kubernetes cluster
4. `eksctl` - create and manage EKS clusters
5. `AWS CLI` - configure AWS credentials and interact with AWS services

### Remote Resources
1. AWS CodeBuild - build Docker images remotely
2. AWS ECR - host Docker images
3. AWS EKS - run the application in Kubernetes
4. AWS CloudWatch - monitor logs and activity
5. GitHub - source code repository

## Build Process
The application is containerized using a `python:3.11-slim` base image. The `buildspec.yaml` in the root directory defines the CI/CD pipeline. Any push to the `main` branch triggers CodeBuild to:
1. Authenticate with ECR
2. Build the Docker image from `analytics/Dockerfile`
3. Tag it with `latest` and `$CODEBUILD_BUILD_NUMBER` for semantic versioning (e.g. `1.0.1`)
4. Push both tags to ECR

## Deployment Process
All Kubernetes manifests are in the `deployment/` directory:
- `pv.yaml` / `pvc.yaml` - persistent storage for PostgreSQL
- `postgresql-deployment.yaml` / `postgresql-service.yaml` - database layer
- `configmap.yaml` - plaintext environment variables (DB host, port, name, user) and Secret (DB password)
- `coworking.yaml` - application deployment and LoadBalancer service

Apply in this order:
```bash
kubectl apply -f pvc.yaml
kubectl apply -f pv.yaml
kubectl apply -f postgresql-deployment.yaml
kubectl apply -f postgresql-service.yaml
kubectl apply -f configmap.yaml
kubectl apply -f coworking.yaml
```

## Releasing New Builds
To release a new version, push updated code to the `main` branch. CodeBuild automatically builds and tags the new image. Update the image tag in `coworking.yaml` to the new version number and run:
```bash
kubectl apply -f deployment/coworking.yaml
```
Kubernetes performs a rolling update with zero downtime.

## Stand Out Suggestions

### Instance Type
`t3.small` is used for this project. For production, `t3.medium` would be recommended as it provides more memory for handling concurrent API requests and database connections without throttling.

### Cost Savings
Delete the EKS cluster when not actively in use with `eksctl delete cluster --name my-cluster --region us-east-1` since EC2 nodes incur hourly charges even when idle. Using Spot Instances for the node group can reduce compute costs by up to 70% for non-critical workloads.

### Resource Allocation
The deployment is configured with reasonable CPU and memory limits to prevent any single pod from consuming excessive cluster resources, ensuring stable performance across all services.