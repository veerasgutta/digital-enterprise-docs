# 🚀 Digital Enterprise Platform - Open Source Setup Guide

This guide will help you transform the Digital Enterprise Platform into a proper open source project with separate repositories, Terraform infrastructure, and a demo platform.

## 📋 Phase 1: Repository Setup (Under `veerasgutta` Organization)

### 1.1 Create GitHub Organization
```bash
# First, create the GitHub organization 'veerasgutta' if it doesn't exist
# https://github.com/organizations/new
```

### 1.2 Repository Structure
Create these 7 repositories under the organization (starting as **private**):

```bash
# Core platform repository (private initially)
gh repo create veerasgutta/digital-enterprise-core --private
gh repo create veerasgutta/enterprise-product-agent --private
gh repo create veerasgutta/enterprise-sales-agent --private
gh repo create veerasgutta/enterprise-marketing-agent --private
gh repo create veerasgutta/enterprise-infrastructure --private
gh repo create veerasgutta/enterprise-demo-platform --private
gh repo create veerasgutta/enterprise-extensions --private
```

### 1.3 Private Development Phase Strategy
```bash
# Phase 1: Private Development (Current)
# - Develop and test all components
# - Set up proper CI/CD pipelines
# - Create comprehensive documentation
# - Build demo platform
# - Security review and hardening

# Phase 2: Beta Testing (Invite-only)
# - Invite trusted developers for testing
# - Gather feedback and iterate
# - Performance optimization
# - Bug fixes and improvements

# Phase 3: Public Launch (When ready)
# - Make repositories public
# - Community announcement
# - Marketing and promotion
# - Open source community building
```

## 🏗️ Phase 2: Infrastructure Setup

### 2.1 AWS Prerequisites
```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS credentials
aws configure
```

### 2.2 Terraform State Backend
```bash
# Create S3 bucket for Terraform state
aws s3 mb s3://digital-enterprise-terraform-state --region us-west-2

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket digital-enterprise-terraform-state \
    --versioning-configuration Status=Enabled

# Create DynamoDB table for state locking
aws dynamodb create-table \
    --table-name terraform-state-lock \
    --attribute-definitions AttributeName=LockID,AttributeType=S \
    --key-schema AttributeName=LockID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region us-west-2
```

### 2.3 Deploy Infrastructure
```bash
# Clone the infrastructure repository
git clone https://github.com/veerasgutta/enterprise-infrastructure.git
cd enterprise-infrastructure

# Deploy development environment
chmod +x scripts/deploy.sh
./scripts/deploy.sh dev plan
./scripts/deploy.sh dev apply

# Or using PowerShell on Windows
.\scripts\deploy.ps1 dev plan
.\scripts\deploy.ps1 dev apply
```

## 🐳 Phase 3: Container Setup

### 3.1 Build Docker Images
```bash
# Build core platform image
docker build -t digital-enterprise-core:latest -f Dockerfile.core .

# Build individual agent images
docker build -t enterprise-product-agent:latest -f agents/product/Dockerfile .
docker build -t enterprise-sales-agent:latest -f agents/sales/Dockerfile .
docker build -t enterprise-marketing-agent:latest -f agents/marketing/Dockerfile .

# Push to ECR
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-west-2.amazonaws.com
docker tag digital-enterprise-core:latest <account-id>.dkr.ecr.us-west-2.amazonaws.com/digital-enterprise-core:latest
docker push <account-id>.dkr.ecr.us-west-2.amazonaws.com/digital-enterprise-core:latest
```

### 3.2 Kubernetes Deployment
```bash
# Configure kubectl
aws eks update-kubeconfig --region us-west-2 --name digital-enterprise-dev

# Deploy applications
kubectl apply -f kubernetes/namespace.yaml
kubectl apply -f kubernetes/core/
kubectl apply -f kubernetes/agents/
kubectl apply -f kubernetes/monitoring/
```

## 🌐 Phase 4: Demo Platform Setup

### 4.1 Deploy Demo Platform
```bash
# Clone demo platform repository
git clone https://github.com/veerasgutta/enterprise-demo-platform.git
cd enterprise-demo-platform

# Install dependencies
npm install

# Configure environment
cp .env.example .env
# Edit .env with your AWS and database configurations

# Deploy to production
npm run build
npm run deploy
```

### 4.2 Demo Platform Features
- **Interactive Dashboard**: Real-time enterprise metrics
- **Agent Playground**: Test individual agents
- **Workflow Simulator**: End-to-end business processes
- **API Documentation**: Swagger/OpenAPI specs
- **Video Tutorials**: Getting started guides

## 🔧 Phase 5: Development Setup

### 5.1 Local Development Environment
```bash
# Install prerequisites
# - Node.js 18+
# - Python 3.9+
# - Docker Desktop
# - kubectl
# - helm

# Clone all repositories
mkdir digital-enterprise-workspace
cd digital-enterprise-workspace

git clone https://github.com/veerasgutta/digital-enterprise-core.git
git clone https://github.com/veerasgutta/enterprise-product-agent.git
git clone https://github.com/veerasgutta/enterprise-sales-agent.git
git clone https://github.com/veerasgutta/enterprise-marketing-agent.git
git clone https://github.com/veerasgutta/enterprise-demo-platform.git

# Setup development environment
cd digital-enterprise-core
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# Install pre-commit hooks
pre-commit install
```

### 5.2 Branch Protection and PR Process
```bash
# Configure branch protection for main repositories
gh api repos/veerasgutta/digital-enterprise-core/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["ci/tests","ci/security-scan"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":2}' \
  --field restrictions=null
```

## 📊 Phase 6: Monitoring and Observability

### 6.1 Deploy Monitoring Stack
```bash
# Install Prometheus and Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install monitoring
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values monitoring/prometheus-values.yaml

# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
# Default credentials: admin/prom-operator
```

### 6.2 Logging Setup
```bash
# Install ELK Stack
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch --namespace logging --create-namespace
helm install kibana elastic/kibana --namespace logging
helm install filebeat elastic/filebeat --namespace logging
```

## 🚦 Phase 7: CI/CD Pipeline

### 7.1 GitHub Actions Setup
```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests
        run: pytest tests/
      - name: Security scan
        run: bandit -r src/
  
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to AWS
        run: |
          aws eks update-kubeconfig --region us-west-2 --name digital-enterprise-prod
          kubectl apply -f kubernetes/
```

## 📈 Phase 8: Success Metrics

### 8.1 Demo Platform Analytics
- **User Engagement**: Track demo usage, session duration
- **Feature Adoption**: Monitor which agents are most popular
- **Performance Metrics**: Response times, error rates
- **Business Metrics**: Lead generation, conversion rates

### 8.2 Open Source Community
- **GitHub Stars**: Target 1000+ stars in first 6 months
- **Contributors**: Attract 50+ contributors
- **Issues/PRs**: Maintain healthy issue resolution
- **Documentation**: Comprehensive guides and tutorials

## 🎯 Phase 9: Go-Public Strategy

### 9.1 Pre-Public Checklist
- [ ] All repositories fully functional in private mode
- [ ] Comprehensive documentation and tutorials
- [ ] Demo platform polished and user-friendly
- [ ] Security audit completed
- [ ] Performance benchmarks established
- [ ] Legal review (licenses, compliance)
- [ ] Community guidelines and code of conduct
- [ ] Contributor onboarding process

### 9.2 Making Repositories Public
```bash
# When ready to go public, update repository visibility
gh repo edit veerasgutta/digital-enterprise-core --visibility public
gh repo edit veerasgutta/enterprise-product-agent --visibility public
gh repo edit veerasgutta/enterprise-sales-agent --visibility public
gh repo edit veerasgutta/enterprise-marketing-agent --visibility public
gh repo edit veerasgutta/enterprise-infrastructure --visibility public
gh repo edit veerasgutta/enterprise-demo-platform --visibility public
gh repo edit veerasgutta/enterprise-extensions --visibility public
```

### 9.3 Public Launch Strategy
- [ ] Announce on social media and tech communities
- [ ] Submit to Product Hunt and Hacker News
- [ ] Create launch blog post and press release
- [ ] Set up community channels (Discord/Slack)
- [ ] Engage with early adopters and contributors
- [ ] Monitor feedback and iterate quickly

## 🆘 Troubleshooting

### Common Issues
1. **Terraform State Conflicts**: Use proper state locking
2. **EKS Access Issues**: Check IAM permissions
3. **Database Connections**: Verify security groups
4. **Resource Limits**: Monitor AWS quotas
5. **Demo Platform Errors**: Check application logs

### Support Channels
- **GitHub Discussions**: For community questions
- **Issues**: For bug reports and feature requests
- **Documentation**: Comprehensive guides and FAQs
- **Video Tutorials**: Step-by-step setup guides

## 📚 Resources

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [EKS Best Practices](https://aws.github.io/aws-eks-best-practices/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Open Source Guides](https://opensource.guide/)

---

🚀 **Ready to transform your enterprise with AI-powered automation?** 

Start with our demo platform and scale to full production deployment!
