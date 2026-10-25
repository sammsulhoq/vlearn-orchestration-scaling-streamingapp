# Deployment and Operations Guide

## Environment

| Setting | Value |
|---|---|
| AWS profile | herovired |
| Region | ap-south-1 |
| EKS cluster | herovired-streamingapp |
| Kubernetes namespace | streamingapp |
| Helm release | streamingapp |
| Chart | helm/streamingapp |
| Validated image tag | build-5 |
| Frontend NodePort | 30080 |

Use the herovired profile explicitly for project AWS commands.
The separate ekmaataaisibhi profile is unrelated to this project.

## Prerequisites

The operator requires Git, Docker, AWS CLI v2, eksctl, kubectl 1.35,
and Helm 3. Temporary lab credentials require an access key, secret key,
and session token from the same active session.

Infrastructure prerequisites:
- ECR repositories: streamingapp/frontend, auth, streaming, admin and chat.
- EKS cluster and managed nodes described in infrastructure/eks-cluster.yaml.
- EBS CSI driver with its IAM role.
- AWS Load Balancer Controller, installed during initial provisioning.
- Metrics Server.
- CloudWatch Observability add-on and its IAM role.
- Private S3 media bucket.
- Admin and streaming IAM service accounts with their respective S3 policies.
- Namespace streamingapp and Secret streamingapp-secrets.

The checked-in IAM policy JSON files document application and controller
permissions. IAM service-account roles use the cluster's OIDC provider.

## Initial provisioning sequence

1. Fork the upstream application and clone the fork.
2. Build locally with docker compose build and verify the application.
3. Create the five ECR repositories and push initial images.
4. Provision Jenkins on EC2 with Java 21, Docker and AWS CLI.
5. Give Jenkins an EC2 instance role with project ECR push/pull permissions.
6. Configure a Pipeline job using the root Jenkinsfile and branch main.
7. Validate and create EKS using the configuration below.
8. Install EBS CSI and Metrics Server.
9. Create the namespace, application Secret and S3 service-account roles.
10. Select a successful Jenkins image tag in Helm values.
11. Validate and install the chart.
12. Configure permitted browser origins and NodePort source-IP access.
13. Install CloudWatch monitoring and connect alarms to SNS.

Cluster provisioning commands:

```bash
AWS_PROFILE=herovired eksctl create cluster \
  --config-file infrastructure/eks-cluster.yaml \
  --dry-run

AWS_PROFILE=herovired eksctl create cluster \
  --config-file infrastructure/eks-cluster.yaml
```

Review the configuration's publicAccessCIDRs before provisioning:
it must contain the operator's current public IPv4 address followed by /32.

The lab account rejected AWS load balancer creation. The final chart
therefore uses NodePort. See architecture.md for the restriction.

## Connect to the existing cluster

```bash
aws sts get-caller-identity --profile herovired

aws eks update-kubeconfig \
  --name herovired-streamingapp \
  --region ap-south-1 \
  --profile herovired \
  --alias herovired-streamingapp

kubectl config current-context
kubectl get nodes
```

Expected context: herovired-streamingapp.
Expected baseline: two Ready worker nodes.

## Secret requirements

The existing Secret streamingapp-secrets contains:
- mongodb-root-username
- mongodb-root-password
- MONGO_URI
- JWT_SECRET

MONGO_URI points to mongo:27017, database streamingapp, with
authSource=admin and credentials matching MongoDB.

Generate fresh random credentials for a new environment. Do not put
Secret manifests, exported Secret data or credentials in Git.

MongoDB root credentials initialize the database only on an empty disk.
Changing the Secret alone does not change an existing database password.
A rotation must update the database and Secret together, then restart
the affected workloads.

## Validate and deploy

Run from the repository root:

```bash
helm lint helm/streamingapp

helm template streamingapp helm/streamingapp \
  --namespace streamingapp |
kubectl apply --namespace streamingapp --dry-run=server -f -
```

For a new release:

```bash
helm install streamingapp helm/streamingapp \
  --namespace streamingapp --wait --timeout 15m
```

For the existing release:

```bash
helm upgrade streamingapp helm/streamingapp \
  --namespace streamingapp --wait --timeout 10m
```

Update image.tag in values.yaml before deploying a new image set.
Keep Chart.yaml appVersion aligned with the selected application version.
A Git push builds images through Jenkins but does not deploy them to EKS.

## Verify deployment

```bash
helm status streamingapp -n streamingapp
kubectl get deployments,pods,services,hpa -n streamingapp
kubectl get pvc -n streamingapp
kubectl top nodes
kubectl top pods -n streamingapp
```

Baseline: nine ready application pods, including MongoDB.
HPA may temporarily increase auth or streaming replicas.
MongoDB's PVC should remain Bound.

## Access the application

Find current worker addresses:

```bash
kubectl get nodes -o wide
```

Open a worker's public IP on port 30080. The security group must allow
TCP 30080 from the operator's current public IP.

clientUrls in Helm values must include the exact browser origin,
including protocol and port. Update it through helm upgrade if addresses
change.

For local troubleshooting:

```bash
kubectl port-forward -n streamingapp service/frontend 8081:80
```

Then open http://localhost:8081 and keep the forwarding process running.

## Functional validation

1. Register and log in using a project test account.
2. Verify an administrator can access the admin dashboard.
3. Upload a short MP4 and a thumbnail.
4. Confirm objects exist under videos/ and thumbnails/ in the private bucket.
5. Confirm catalogue thumbnails, playback and seeking work.
6. Verify playback requests return HTTP 206 with Content-Range.
7. Exchange chat messages between separate browser sessions.
8. Refresh and verify chat history.
9. Replace only the MongoDB pod and verify accounts and video records remain.

Administrator access is based on role=admin and a newly issued login token.
Only an authorized test account should be promoted. Sign out and back in
after changing its database role.

## Scaling verification

Auth and streaming scale between 2 and 4 replicas at a 60% CPU target.

```bash
kubectl get hpa -n streamingapp
kubectl describe hpa streamingapp-auth -n streamingapp
```

The submitted evidence shows auth scaling from 2 to 4 and back to 2.
No worker-node autoscaler is installed.

## Monitoring

Application log group:
/aws/containerinsights/herovired-streamingapp/application

```bash
aws logs tail \
  /aws/containerinsights/herovired-streamingapp/application \
  --since 5m --format short \
  --region ap-south-1 --profile herovired
```

CloudWatch alarms:
- streamingapp-node-cpu-high
- streamingapp-node-memory-high

Both use ContainerInsights metrics with the ClusterName dimension.
Threshold: above 80% for two consecutive one-minute periods.

SNS topic: herovired-streamingapp-alerts.
Confirmed email subscriptions receive ALARM and OK notifications.
SNS email delivery was verified with an explicitly labelled test message.

## Troubleshooting

| Symptom | Check |
|---|---|
| ExpiredToken or Unauthorized | Renew all three lab credentials; refresh kubeconfig |
| Helm timeout | Inspect pods, PVCs and events before retrying |
| MongoDB stays unready | Check probe timeouts and current pod revision |
| Frontend fails to start | Check Nginx logs and nginx -t |
| Browser request rejected by CORS | Check clientUrls against the exact browser origin |
| S3 AccessDenied | Check serviceAccountName, role trust and bucket policy scope |
| No application logs | Check Fluent Bit logs and unfiltered CloudWatch output |
| Jenkins unavailable | Check EC2 state, public IP, security group and Jenkins service |
| NodePort unavailable | Check current worker IPs and the source-IP firewall rule |

Useful diagnostics:

```bash
kubectl get events -n streamingapp \
  --sort-by=.metadata.creationTimestamp

kubectl logs deployment/streamingapp-admin \
  -n streamingapp --since=10m --tail=80
```

Never publish credentials, tokens or unredacted connection strings in
logs, screenshots or documentation.

## Cost and teardown

EKS, EC2, EBS, S3 and CloudWatch resources consume the lab budget.

Before teardown, preserve required evidence and any data that must be kept.
The chart's PVC and StorageClass use a deletion policy that can remove
the database disk during uninstall. Helm rollback is not a database backup.

Inventory project resources before deleting them. Do not delete the
existing default VPC, shared subnets or unrelated resources.
Application cleanup, EKS deletion and Jenkins termination are separate
operations. Check for remaining EBS volumes, load balancer resources,
ECR images, S3 objects, CloudWatch resources and SNS subscriptions afterward.
