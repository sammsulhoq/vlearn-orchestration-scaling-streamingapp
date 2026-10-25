# StreamingApp — Orchestration and Scaling

Hero Vired graded project deploying a MERN streaming application on
Amazon EKS using Docker, Amazon ECR, Jenkins and Helm.

Upstream application:
[UnpredictablePrashant/StreamingApp](https://github.com/UnpredictablePrashant/StreamingApp)

## Implemented

- Five Docker images: React frontend and four Node.js backend services.
- Private Amazon ECR repositories.
- Jenkins on EC2 with automatic Git polling, image builds and ECR pushes.
- Helm deployment to EKS 1.35 with two managed worker nodes.
- Nginx API routing, React route fallback and WebSocket support.
- MongoDB StatefulSet with encrypted EBS gp3 persistent storage.
- Private S3 media storage with separate admin and streaming IAM roles.
- JWT authentication and administrator access.
- Video uploads, thumbnail delivery, playback and seeking.
- Real-time chat and persisted message history.
- CPU-based HPA for auth and streaming, from two to four replicas.
- CloudWatch container logs, infrastructure metrics and alarms.
- SNS email notifications for alarm and recovery states.

## Documentation

| Document | Contents |
|---|---|
| [Architecture](docs/architecture.md) | Diagram, components, routing, security and limitations |
| [Deployment and operations](docs/deployment.md) | Provisioning sequence, deployment, verification and troubleshooting |
| [Evidence](docs/evidence.md) | Linked screenshots of implementation and validation |
| [Upstream README](docs/upstream-readme.md) | Original application documentation |

## Repository layout

```text
backend/
  adminService/
  authService/
  chatService/
  streamingService/
frontend/
helm/streamingapp/
infrastructure/
docs/
  screenshots/
Jenkinsfile
docker-compose.yml
```

## Build and deployment

Jenkins polls main approximately every five minutes. A changed commit
triggers builds and pushes all five images using build-number tags.

EKS deployments are performed manually with Helm. Select the desired
image.tag in helm/streamingapp/values.yaml, then run:

```bash
helm lint helm/streamingapp

helm upgrade --install streamingapp helm/streamingapp \
  --namespace streamingapp \
  --wait \
  --timeout 15m
```

Provisioning prerequisites, namespace and Secret requirements are documented
in the deployment guide. The validated application image tag is build-5.

## Validation results

| Test | Result |
|---|---|
| Registration and login on EKS | Passed |
| Administrator upload to private S3 | Passed |
| Thumbnail display and video playback | Passed |
| Seeking with HTTP 206 range responses | Passed |
| Chat between separate browser sessions | Passed |
| Chat history after refresh | Passed |
| MongoDB data after pod replacement | Passed |
| HPA scale-up from 2 to 4 and scale-down to 2 | Passed |
| CloudWatch application log delivery | Passed |
| SNS email delivery | Passed |

The HPA test used a temporary CPU workload. It demonstrates autoscaler
behavior, not application throughput capacity.

## Access and lab constraints

AWS rejected load balancer creation for this account. The frontend uses
NodePort 30080 with source-IP-restricted security-group access instead.

Validated endpoints at the time of testing:
- http://13.201.183.231:30080
- http://13.201.166.45:30080

Worker public IPs can change. Access is restricted to the configured
operator IP, so these are not unrestricted public demo links.

The lab uses HTTP, one MongoDB instance, and one chat replica.
Worker-node autoscaling is not installed. Production considerations and
the unresolved dependency audit findings are recorded in the architecture
document.

## Credentials and configuration

AWS CLI project commands use profile herovired in ap-south-1.
Application S3 access uses IAM roles for service accounts.

Database credentials and JWT secrets are stored outside Git in a
Kubernetes Secret. Database connection logging was corrected and the
exposed database password was rotated.

Do not commit .env files, AWS credentials, private keys or exported
Kubernetes Secret data.

## Optional bonus

SNS email notifications are implemented. Slack, Microsoft Teams or
Telegram integration is not implemented.
