# Genii AI Production Deployment Guide

Comprehensive guide for deploying Genii AI to production environments.

## Table of Contents

1. [Deployment Options](#deployment-options)
2. [Prerequisites](#prerequisites)
3. [Kubernetes Deployment](#kubernetes-deployment)
4. [AWS Deployment](#aws-deployment)
5. [GCP Deployment](#gcp-deployment)
6. [Post-Deployment](#post-deployment)
7. [Scaling](#scaling)
8. [Disaster Recovery](#disaster-recovery)

## Deployment Options

### Option 1: Kubernetes (Recommended)
- Most flexible and scalable
- Works on any cloud provider
- Self-healing and auto-scaling
- Cost: Variable based on usage

### Option 2: AWS ECS/Fargate
- Fully managed container orchestration
- Good AWS integration
- Simpler than Kubernetes
- Cost: ~$200-500/month

### Option 3: Docker Compose (Not Recommended for Production)
- Simple single-server deployment
- No auto-scaling or failover
- Good for staging/testing
- Cost: ~$50-100/month

## Prerequisites

### Required Accounts & Services

1. **Cloud Provider Account**
   - AWS, GCP, or Azure
   - Payment method configured
   - IAM permissions set up

2. **Domain & DNS**
   - Domain name registered
   - DNS hosted (Route53, CloudFlare, etc.)
   - SSL certificates (Let's Encrypt recommended)

3. **Third-Party Services**
   - OpenAI API access
   - ERPNext instance (Frappe Cloud or self-hosted)
   - SMTP service (SendGrid, AWS SES)
   - Slack workspace (optional)

4. **Development Tools**
   - kubectl (Kubernetes CLI)
   - helm (Package manager for Kubernetes)
   - docker (Container runtime)
   - awscli or gcloud (Cloud provider CLI)

## Kubernetes Deployment

### Step 1: Cluster Setup

#### Using AWS EKS

```bash
# Install eksctl
brew install eksctl  # macOS
# or download from https://eksctl.io

# Create cluster
eksctl create cluster \
  --name genii-prod \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 10 \
  --managed

# Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name genii-prod
```

#### Using GKE

```bash
# Create cluster
gcloud container clusters create genii-prod \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type n1-standard-2 \
  --enable-autoscaling \
  --min-nodes 2 \
  --max-nodes 10

# Get credentials
gcloud container clusters get-credentials genii-prod --zone us-central1-a
```

### Step 2: Install Dependencies

```bash
# Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

# Install cert-manager for SSL
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Install metrics server for autoscaling
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Step 3: Configure Secrets

```bash
# Create namespace
kubectl create namespace genii-prod

# Create database secret
kubectl create secret generic db-credentials \
  --from-literal=username=genii_admin \
  --from-literal=password=YOUR_DB_PASSWORD \
  --from-literal=database=genii_prod \
  -n genii-prod

# Create ERPNext credentials
kubectl create secret generic erpnext-credentials \
  --from-literal=url=https://your-erpnext.frappe.cloud \
  --from-literal=api-key=YOUR_API_KEY \
  --from-literal=api-secret=YOUR_API_SECRET \
  -n genii-prod

# Create AI service keys
kubectl create secret generic ai-keys \
  --from-literal=openai-key=sk-YOUR_OPENAI_KEY \
  --from-literal=anthropic-key=sk-ant-YOUR_ANTHROPIC_KEY \
  -n genii-prod

# Create JWT secret
kubectl create secret generic jwt-secret \
  --from-literal=secret=$(openssl rand -base64 32) \
  -n genii-prod
```

### Step 4: Deploy Application

```bash
# Clone repository
git clone https://github.com/davidmschy/genii-ai-prod.git
cd genii-ai-prod

# Update k8s manifests with your values
vim k8s/deployment.yaml
vim k8s/ingress.yaml

# Apply manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
kubectl apply -f k8s/hpa.yaml  # Horizontal Pod Autoscaler

# Verify deployment
kubectl get pods -n genii-prod
kubectl logs -f deployment/genii-api -n genii-prod
```

### Step 5: Configure SSL

```bash
# Apply ClusterIssuer for Let's Encrypt
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: david@geniinow.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF

# SSL will be automatically provisioned via cert-manager
```

### Step 6: Database Setup

```bash
# Run migrations
kubectl run migration \
  --image=davidmschy/genii-ai-prod:latest \
  --restart=Never \
  --command -- npm run db:migrate \
  -n genii-prod

# Check migration logs
kubectl logs migration -n genii-prod

# Clean up
kubectl delete pod migration -n genii-prod
```

## AWS Deployment

### Using AWS ECS Fargate

```bash
# Install AWS CLI and configure
aws configure

# Create ECS cluster
aws ecs create-cluster --cluster-name genii-prod

# Create task definition
aws ecs register-task-definition --cli-input-json file://aws/task-definition.json

# Create service
aws ecs create-service \
  --cluster genii-prod \
  --service-name genii-api \
  --task-definition genii-api:1 \
  --desired-count 3 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}"

# Create load balancer
aws elbv2 create-load-balancer \
  --name genii-prod-lb \
  --subnets subnet-xxx subnet-yyy \
  --security-groups sg-xxx
```

### RDS Database

```bash
# Create PostgreSQL RDS instance
aws rds create-db-instance \
  --db-instance-identifier genii-prod-db \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --engine-version 15.4 \
  --master-username genii_admin \
  --master-user-password YOUR_PASSWORD \
  --allocated-storage 100 \
  --storage-type gp3 \
  --backup-retention-period 7 \
  --multi-az \
  --publicly-accessible false
```

### ElastiCache Redis

```bash
# Create Redis cluster
aws elasticache create-replication-group \
  --replication-group-id genii-prod-redis \
  --replication-group-description "Genii Production Redis" \
  --engine redis \
  --cache-node-type cache.t3.medium \
  --num-cache-clusters 2 \
  --automatic-failover-enabled
```

## GCP Deployment

### Using Cloud Run

```bash
# Build and push container
gcloud builds submit --tag gcr.io/PROJECT_ID/genii-ai-prod

# Deploy to Cloud Run
gcloud run deploy genii-api \
  --image gcr.io/PROJECT_ID/genii-ai-prod \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --min-instances 2 \
  --max-instances 100 \
  --cpu 2 \
  --memory 4Gi \
  --set-env-vars "NODE_ENV=production" \
  --set-secrets "DATABASE_URL=db-url:latest,ERPNEXT_API_KEY=erpnext-key:latest"
```

### Cloud SQL

```bash
# Create PostgreSQL instance
gcloud sql instances create genii-prod-db \
  --database-version=POSTGRES_15 \
  --tier=db-custom-4-16384 \
  --region=us-central1 \
  --backup \
  --backup-start-time=03:00

# Create database
gcloud sql databases create genii_prod --instance=genii-prod-db
```

## Post-Deployment

### Verify Deployment

```bash
# Check service health
curl https://api.genii.example.com/health

# Check database connection
curl https://api.genii.example.com/health/db

# Check Redis connection
curl https://api.genii.example.com/health/redis

# Run smoke tests
npm run test:smoke -- --url=https://api.genii.example.com
```

### Configure Monitoring

```bash
# Install Prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring

# Access Grafana
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
# Username: admin, Password: prom-operator
```

### Set Up Logging

```bash
# Install ELK stack
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch -n logging
helm install kibana elastic/kibana -n logging
helm install filebeat elastic/filebeat -n logging
```

### Configure Backups

```bash
# PostgreSQL automated backups (AWS RDS)
aws rds modify-db-instance \
  --db-instance-identifier genii-prod-db \
  --backup-retention-period 30 \
  --preferred-backup-window "03:00-04:00"

# Manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier genii-prod-db \
  --db-snapshot-identifier genii-prod-$(date +%Y%m%d)
```

## Scaling

### Horizontal Pod Autoscaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: genii-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: genii-api
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Database Scaling

```bash
# Scale up RDS instance
aws rds modify-db-instance \
  --db-instance-identifier genii-prod-db \
  --db-instance-class db.m5.xlarge \
  --apply-immediately

# Add read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier genii-prod-db-replica \
  --source-db-instance-identifier genii-prod-db
```

## Disaster Recovery

### Backup Strategy

- **Database**: Daily automated backups, 30-day retention
- **Configuration**: Stored in version control (Git)
- **Secrets**: Backed up in encrypted vault
- **File Storage**: S3 with versioning enabled

### Recovery Procedure

1. **Database Recovery**:
```bash
# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier genii-prod-db-restored \
  --db-snapshot-identifier genii-prod-20260219
```

2. **Application Rollback**:
```bash
# Rollback to previous version
kubectl rollout undo deployment/genii-api -n genii-prod

# Or rollback to specific revision
kubectl rollout undo deployment/genii-api --to-revision=2 -n genii-prod
```

3. **Full Site Recovery**:
```bash
# Restore entire cluster from backup
eksctl create cluster --config-file=backup/cluster-config.yaml
kubectl apply -f backup/manifests/
```

## Troubleshooting

### Pod Not Starting

```bash
# Check pod status
kubectl describe pod POD_NAME -n genii-prod

# Check logs
kubectl logs POD_NAME -n genii-prod --previous

# Check events
kubectl get events -n genii-prod --sort-by='.lastTimestamp'
```

### High Memory Usage

```bash
# Check memory metrics
kubectl top pods -n genii-prod

# Increase memory limit
kubectl set resources deployment genii-api --limits=memory=4Gi -n genii-prod
```

### Database Connection Issues

```bash
# Test connection from pod
kubectl run -it --rm debug --image=postgres:15 --restart=Never -- psql $DATABASE_URL

# Check security groups (AWS)
aws ec2 describe-security-groups --group-ids sg-xxx
```

## Security Checklist

- [ ] SSL/TLS certificates configured
- [ ] Database encryption at rest enabled
- [ ] Secrets stored in secure vault (not in code)
- [ ] Network policies configured
- [ ] RBAC roles properly defined
- [ ] Pod security policies enforced
- [ ] Regular security updates applied
- [ ] Intrusion detection enabled
- [ ] Audit logging configured
- [ ] Backup encryption enabled

## Cost Optimization

### Right-Sizing

```bash
# Analyze resource usage
kubectl top pods -n genii-prod --sort-by=memory

# Adjust resource requests/limits
kubectl set resources deployment genii-api \
  --requests=cpu=500m,memory=1Gi \
  --limits=cpu=1000m,memory=2Gi \
  -n genii-prod
```

### Cost Monitoring

- Enable cloud provider cost alerts
- Use spot instances for non-critical workloads
- Schedule dev/staging environments to shut down at night
- Use reserved instances for predictable workloads

---

**Need Help?** Contact david@geniinow.com
