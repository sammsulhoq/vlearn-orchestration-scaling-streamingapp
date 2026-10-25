# Project Evidence

Screenshots capture the system at different stages. Earlier images may
show previous image tags, IP addresses or configurations.

## Version control and local containers

- [GitHub fork](screenshots/01-github-fork.png)
- [Local repository setup](screenshots/02-local-repository-setup.png)
- [Docker images](screenshots/03-local-docker-images.png)
- [Running local containers](screenshots/04-local-containers-running.png)
- [Local homepage](screenshots/05-local-homepage.png)
- [Local login](screenshots/06-local-login-success.png)
- [Frontend route refresh](screenshots/07-frontend-route-refresh.png)
- [Docker configuration commit](screenshots/08-docker-setup-commit.png)

## Amazon ECR

- [ECR repositories](screenshots/09-ecr-repositories.png)
- Initial image uploads: [Part A](screenshots/10a-ecr-images-pushed.png), [Part B](screenshots/10b-ecr-images-pushed.png)
- Jenkins image uploads: [Part A](screenshots/20a-jenkins-ecr-images.png), [Part B](screenshots/20b-jenkins-ecr-images.png), [Part C](screenshots/20c-jenkins-ecr-images.png)

## Jenkins CI

- [Jenkins security group](screenshots/11-jenkins-security-group.png)
- [Jenkins EC2 instance](screenshots/12-jenkins-ec2-instance.png)
- [SSH access](screenshots/13-jenkins-server-ssh.png)
- [Jenkins service](screenshots/14-jenkins-service-running.png)
- [Jenkins dashboard](screenshots/15-jenkins-dashboard.png)
- [Docker test as Jenkins user](screenshots/16-jenkins-docker-test.png)
- [AWS CLI installation](screenshots/17-jenkins-aws-cli.png)
- [IAM role access](screenshots/18-jenkins-aws-role-access.png)
- [Successful pipeline](screenshots/19-jenkins-pipeline-success.png)
- [Automatic SCM trigger](screenshots/21a-jenkins-auto-trigger.png)
- [Automatic build success](screenshots/21b-jenkins-auto-build-success.png)
- [Deployment-ready image build](screenshots/26-eks-ready-images-build.png)

## EKS, Helm and storage

- [Kubernetes tooling](screenshots/22-kubernetes-tools-installed.png)
- [Cluster creation](screenshots/23a-eks-cluster-created.png)
- [Active EKS cluster](screenshots/23b-eks-cluster-active.png)
- [Ready worker nodes](screenshots/24-eks-nodes-ready.png)
- [Nginx authentication routing](screenshots/25-nginx-auth-routing.png)
- [EBS CSI driver](screenshots/27-ebs-csi-driver-ready.png)
- [Secret metadata only](screenshots/28-kubernetes-secret-created.png)
- [Load Balancer Controller](screenshots/29-load-balancer-controller-ready.png)
- [Server-side chart validation](screenshots/30-helm-server-validation.png)
- [Application pods](screenshots/31b-application-pods.png)
- [EKS homepage through port forwarding](screenshots/32-eks-homepage-port-forward.png)
- [EKS login](screenshots/33-eks-login-success.png)
- [AWS account load balancer restriction](screenshots/34-aws-load-balancer-restriction.png)
- [Public NodePort homepage](screenshots/36-eks-public-nodeport-homepage.png)
- [Public login](screenshots/37-eks-public-login-success.png)
- [Build-4 deployment checkpoint](screenshots/40-build-4-deployed.png)
- [Helm and infrastructure commit](screenshots/54-helm-infrastructure-commit.png)

The load balancer restriction led to the documented NodePort substitution.
The build-4 image screenshot is an earlier checkpoint; the final configured
application image tag is build-5.

## S3 and functional validation

- [Private S3 bucket](screenshots/38-s3-private-bucket.png)
- [Application S3 IAM roles](screenshots/39-application-s3-roles.png)
- [Administrator dashboard](screenshots/41-eks-admin-dashboard.png)
- [Uploaded video](screenshots/42a-admin-video-uploaded.png)
- [Media objects in S3](screenshots/42b-s3-media-objects.png)
- [Video playback](screenshots/43a-video-playback.png)
- [HTTP range request](screenshots/43b-video-range-request.png)
- [Chat across two browser sessions](screenshots/44-live-chat-two-sessions.png)

## Scaling and persistence

- [Resource metrics](screenshots/45-kubernetes-resource-metrics.png)
- [Configured HPAs](screenshots/46-hpa-configured.png)
- [Auth scaling to four replicas](screenshots/47a-hpa-scale-up.png)
- [Auth returning to two replicas](screenshots/47b-hpa-scale-down.png)
- [Database before pod replacement](screenshots/53a-mongodb-before-replacement.png)
- [Database after pod replacement](screenshots/53b-mongodb-after-replacement.png)

## Monitoring and notifications

- [CloudWatch add-on](screenshots/48-cloudwatch-addon-ready.png)
- [Centralized application logs](screenshots/49-cloudwatch-application-logs.png)
- [CPU and memory alarms](screenshots/50-cloudwatch-alarms.png)
- [Confirmed SNS subscription](screenshots/51-sns-email-subscription.png)
- [SNS email delivery test](screenshots/52-sns-test-email.png)

SNS email notifications are implemented. Slack, Teams or Telegram ChatOps
integration is not included in this evidence.
