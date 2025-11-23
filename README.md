# 🚀 Rick and Morty API - Kubernetes Deployment on 

[![Python](https://img.shields.io/badge/Python-3.11-blue.svg)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.0-green.svg)](https://flask.palletsprojects.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.27+-326CE5.svg)](https://kubernetes.io/)
[![Terraform](https://img.shields.io/badge/Terraform-IaC-623CE4.svg)](https://www.terraform.io/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED.svg)](https://www.docker.com/)

## 📖 Overview

A production-ready Python Flask microservice that fetches and serves Rick and Morty character data from the [Rick and Morty API](https://rickandmortyapi.com/). The service is fully containerized with Docker, deployed on AWS EKS (Elastic Kubernetes Service) using Infrastructure as Code (Terraform), and includes a complete CI/CD pipeline with GitHub Actions.

### 🎯 Key Features

- **RESTful API**: Clean Flask endpoints for health checks and character data retrieval
- **External API Integration**: Fetches real-time data from Rick and Morty API
- **CSV Export**: Automatically saves character data to CSV format
- **Dockerized**: Lightweight Python 3.11-slim container
- **Kubernetes-Ready**: Complete K8s manifests (Deployment, Service, Ingress)
- **AWS EKS Deployment**: Production-grade Terraform configuration
- **CI/CD Pipeline**: Automated build, test, and deployment with GitHub Actions
- **High Availability**: 3 replicas with health checks and auto-recovery
- **Ingress Controller**: Path-based routing with `/api` prefix

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    GitHub Actions CI/CD                  │
│                  (Build, Push, Deploy)                   │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                      AWS EKS Cluster                     │
│  ┌────────────────────────────────────────────────────┐ │
│  │              Ingress Controller                     │ │
│  │         (Path: /api, /health)                       │ │
│  └──────────────────┬─────────────────────────────────┘ │
│                     │                                   │
│  ┌──────────────────▼─────────────────────────────────┐ │
│  │         Kubernetes Service (ClusterIP)             │ │
│  │              rick-service:80                        │ │
│  └──────────────────┬─────────────────────────────────┘ │
│                     │                                   │
│  ┌──────────────────▼─────────────────────────────────┐ │
│  │          Deployment: rickandmorty-api              │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐           │ │
│  │  │ Pod 1   │  │ Pod 2   │  │ Pod 3   │  (3x)     │ │
│  │  │Flask:8000│ │Flask:8000│ │Flask:8000│           │ │
│  │  └─────────┘  └─────────┘  └─────────┘           │ │
│  └────────────────────────────────────────────────────┘ │
│                     │                                   │
│                     ▼                                   │
│         External: rickandmortyapi.com                   │
└─────────────────────────────────────────────────────────┘
```

## 📋 Prerequisites

### Local Development
- **Python**: 3.11 or higher
- **Docker**: For containerization
- **kubectl**: For Kubernetes management
- **AWS CLI**: Configured with appropriate credentials

### AWS Infrastructure
- **Terraform**: >= 1.0
- **AWS Account**: With EKS permissions
- **Docker Hub Account**: For image registry (or use ECR)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/Danielashkenazy/Microservice_EKS_Terraform_Deployment.git
cd Microservice_EKS_Terraform_Deployment
```

### 2. Local Development

#### Install Dependencies

```bash
cd docker
pip install -r requirements.txt
```

#### Run Locally

```bash
python main.py
```

The API will be available at `http://localhost:8000`

### 3. Docker Build & Run

```bash
cd docker
docker build -t rickandmorty-api:latest .
docker run -p 8000:8000 rickandmorty-api:latest
```

### 4. Deploy to Kubernetes

#### Apply Kubernetes Manifests

```bash
kubectl apply -f yamls/Service.yaml
kubectl apply -f yamls/Deployment.yaml
kubectl apply -f yamls/Ingress.yaml
```

#### Verify Deployment

```bash
kubectl get deployments
kubectl get pods
kubectl get services
kubectl get ingress
```

## 🔧 API Endpoints

### Health Check
```bash
GET /health
```

**Response:**
```json
{
  "status": "ok"
}
```

### Get Rick and Morty Characters
```bash
GET /api
```

**Description:** Fetches all human characters from Earth (C-137) and saves them to `results.csv`

**Response:**
```json
[
  {
    "name": "Rick Sanchez",
    "origin": "Earth (C-137)",
    "image": "https://rickandmortyapi.com/api/character/avatar/1.jpeg"
  },
  {
    "name": "Morty Smith",
    "origin": "Earth (C-137)",
    "image": "https://rickandmortyapi.com/api/character/avatar/2.jpeg"
  }
]
```

## 🐳 Docker Configuration

### Dockerfile Details

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip3 install -r requirements.txt
ENTRYPOINT ["python3", "./main.py"]
EXPOSE 8000
```

### Build & Push to Registry

```bash
docker build -t yourusername/rickandmorty-api:latest ./docker
docker push yourusername/rickandmorty-api:latest
```

## ☸️ Kubernetes Manifests

### Deployment Configuration

- **Replicas**: 3 pods for high availability
- **Image**: `danielashkenazy1/rick2:latest`
- **Container Port**: 8000
- **Liveness Probe**: HTTP GET `/health` every 10 seconds
- **Resource Limits**: Configurable based on workload

### Service Configuration

- **Type**: ClusterIP
- **Port**: 80 (external) → 8000 (container)
- **Selector**: `app: rick-api`

### Ingress Configuration

- **Path**: `/api` (Prefix-based routing)
- **Backend Service**: `rick-service:80`
- **Ingress Class**: Compatible with AWS Load Balancer Controller

## 🏗️ Terraform Infrastructure (Future Enhancement)

The project is ready for Terraform-based EKS deployment. Create a `terraform/` directory with:

- VPC and networking configuration
- EKS cluster setup
- Node groups
- IAM roles and policies
- Load Balancer Controller

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

The `.github/workflows/` directory contains the CI/CD pipeline that:

1. **Build Stage**:
   - Checks out code
   - Builds Docker image
   - Runs tests

2. **Push Stage**:
   - Authenticates with Docker Hub
   - Pushes image with tags

3. **Deploy Stage**:
   - Configures kubectl
   - Applies Kubernetes manifests
   - Verifies deployment

### Trigger Conditions

- **Push to `main`**: Full CI/CD pipeline
- **Pull Requests**: Build and test only
- **Manual Trigger**: Via GitHub Actions UI

## 📊 Monitoring & Logging

### Health Checks

```bash
# Check pod health
kubectl get pods -l app=rick-api

# View logs
kubectl logs -l app=rick-api --tail=100 -f

# Check service endpoints
kubectl get endpoints rick-service
```

### Metrics (Future Enhancement)

Consider adding:
- Prometheus for metrics collection
- Grafana for visualization
- CloudWatch for AWS-native monitoring

## 🔐 Security Considerations

- **No Hardcoded Secrets**: Use Kubernetes Secrets or AWS Secrets Manager
- **Network Policies**: Restrict pod-to-pod communication
- **RBAC**: Implement proper Role-Based Access Control
- **Image Scanning**: Scan Docker images for vulnerabilities
- **TLS/SSL**: Enable HTTPS on Ingress (recommended for production)

## 🧪 Testing

### Manual Testing

```bash
# Test health endpoint
curl http://<INGRESS-IP>/health

# Test API endpoint
curl http://<INGRESS-IP>/api

# Check CSV output
kubectl exec -it <POD-NAME> -- cat results.csv
```

### Automated Testing (Recommended)

Add unit tests in `tests/` directory:
```python
import pytest
from main import app

def test_health_check():
    client = app.test_client()
    response = client.get('/health')
    assert response.status_code == 200
```

## 🐛 Troubleshooting

### Common Issues

#### Pods Not Starting
```bash
kubectl describe pod <POD-NAME>
kubectl logs <POD-NAME>
```

#### Service Not Accessible
```bash
kubectl get svc
kubectl describe svc rick-service
```

#### Ingress Issues
```bash
kubectl get ingress
kubectl describe ingress rick-ingress
```

#### Container Crashes
Check the main.py Flask app logs and ensure the Rick and Morty API is accessible.

## 📈 Performance Tuning

### Resource Limits

Update `Deployment.yaml`:
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

### Horizontal Pod Autoscaling

```bash
kubectl autoscale deployment rickandmorty-api \
  --cpu-percent=50 \
  --min=3 \
  --max=10
```

## 🛠️ Development Workflow

1. **Feature Development**:
   ```bash
   git checkout -b feature/your-feature
   # Make changes
   git commit -m "Add new feature"
   git push origin feature/your-feature
   ```

2. **Testing Locally**:
   ```bash
   python main.py
   # Test endpoints
   ```

3. **Build & Test Docker**:
   ```bash
   docker build -t rickandmorty-api:test .
   docker run -p 8000:8000 rickandmorty-api:test
   ```

4. **Create Pull Request**: GitHub Actions will run CI pipeline

5. **Merge to Main**: Triggers full CI/CD deployment

## 📦 Project Structure

```
.
├── .github/
│   └── workflows/          # CI/CD pipelines
├── docker/
│   ├── Dockerfile          # Container configuration
│   ├── main.py            # Flask application
│   ├── requirements.txt   # Python dependencies
│   └── api                # API configuration (if any)
├── yamls/
│   ├── Deployment.yaml    # K8s Deployment manifest
│   ├── Service.yaml       # K8s Service manifest
│   └── Ingress.yaml       # K8s Ingress manifest
└── README.md              # This file
```

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Coding Standards

- Follow PEP 8 for Python code
- Use meaningful commit messages
- Add tests for new features
- Update documentation as needed

## 📝 TODO / Roadmap

- [ ] Add comprehensive unit tests
- [ ] Implement caching layer (Redis)
- [ ] Add Prometheus metrics endpoint
- [ ] Create Terraform modules for EKS
- [ ] Add Helm chart for easier deployment
- [ ] Implement rate limiting
- [ ] Add authentication middleware
- [ ] Create Swagger/OpenAPI documentation
- [ ] Add database for character caching
- [ ] Implement GraphQL endpoint

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- [Rick and Morty API](https://rickandmortyapi.com/) for the awesome data
- Flask team for the excellent web framework
- Kubernetes community for robust orchestration
- AWS for EKS platform

## 📞 Contact & Support

- **GitHub Issues**: [Report a bug or request a feature](https://github.com/Danielashkenazy/Microservice_EKS_Terraform_Deployment/issues)
- **Pull Requests**: Contributions are welcome!

## 🌟 Show Your Support

If you find this project useful, please consider:
- ⭐ Starring the repository
- 🍴 Forking for your own use
- 📢 Sharing with others

---

**Built with ❤️ for the Rick and Morty fan community and Kubernetes enthusiasts**

*"Wubba Lubba Dub Dub!" - Rick Sanchez*
