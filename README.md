1. Audit access and tighten to least privilege
  
  Audit checklist:
- Cloud IAM: does your CLI user/service account have admin/root access 
  when it only needs read/write to specific resources?
- Database: is the app still connecting as a superuser? (fix from last step)
- Kubernetes: does every pod's ServiceAccount have cluster-admin by default?
- GitHub: does every contributor have admin/write access to main, or 
  should some only have read/PR access?

  json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::placemux-devops-demo-shashank/*"
}

2. Lock down networking and enforce TLS

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-app-only
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: my-app

3. Scan images/dependencies for vulnerabilities and patch


docker scout cves my-app:latest

trivy image my-app:latest

4. Set sensible resource limits/requests
 
 yaml:-

resources:
  requests:        
    cpu: "100m"
    memory: "128Mi"
  limits:          
    cpu: "500m"
    memory: "256Mi"

docker compose:-

services:
  app:
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 256M

5. Identify and remove wasteful resources

docker system df          
docker image prune -a     
docker volume prune       


- Stopped/idle EC2 instances still being billed
- Unused S3 buckets or EBS volumes from earlier testing
- Old container images sitting in your registry
<img width="1402" height="1122" alt="ss0002" src="https://github.com/user-attachments/assets/c794fffe-1e18-40b2-ba67-ad5807301311" />
<img width="1536" height="1024" alt="ss2230" src="https://github.com/user-attachments/assets/f8fb6a2f-fff5-4518-b5c9-8f0a922f0fd3" />

