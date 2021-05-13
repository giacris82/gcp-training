# gcp-training
Practice on Google Cloud Platform

# Main subjects:
- **Public Cloud**: GCP
- **Kubernetes**: GKE
- **CI/CD**: Jenkins
- **Observability and monitoring**: Prometheus_NOTES_- Source code and files must be in a Git repository (choose what you prefer)
### Kubernetes
1. Deploy the simple [python app](src/python/hello) on GKE
2. Make sure the [python app](src/python/hello):    
  - is resilient (hint: it must be restarted if a healthcheck fails)    
  - is reachable: you may use [GKE Ingress](https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer). Hint: it takes a few minutes for the creation
3. Add a new route for the [python app](src/python/hello)
4. Deploy a [traffic generator](src/go/requests-generator) which will call [python app](src/python/hello) to the new route, created at the previous point
### CI/CD- Create CI + CD pipeline(s) for build the Docker images and rollout python and golang apps
### Observability and monitoring- Install Prometheus into GKE (you can use Helm charts provide by community)
- Scrape metrics from python app
- Create a `PromQL` query to get the `NUM_REQUESTS`
## Prerequisites- Install [gcloud command](https://cloud.google.com/sdk/docs/install)
- SDK [initialization](https://cloud.google.com/sdk/docs/initializing)  
- Please choose `europe-west3-b`as default zone and `europe-west3` as default region- Install `kubectl`
- Configure project: `gcloud config set project <YOUR_GCP_PROJECT>`
### Install GKE
```PROJECT_ID=$(gcloud config get-value project)
gcloud services enable container.googleapis.comgcloud container clusters create di-nttdata --machine-type=n1-standard-2 --num-nodes=3 --zone=europe-west3-b --project=$PROJECT_ID
gcloud container clusters get-credentials di-nttdata --zone=europe-west3-b --project=$PROJECT_ID```
### Create Jenkins instance- Create Compute Engine instance
```gcloud beta compute --project=$PROJECT_ID instances create jenkins --zone=europe-west3-b --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --tags=jenkins-instance --image=debian-10-buster-v20210316 --image-project=debian-cloud --boot-disk-size=20GB --boot-disk-type=pd-balanced --boot-disk-device-name=jenkins-boot --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any```
- Grant access to Jenkins UI```gcloud compute firewall-rules --project=$PROJECT_ID create fw-jenkins-http --source-ranges=<YOUR_IP_ADDRESS> --allow=tcp:443,tcp:8080 --direction=IN --network=default --target-tags=jenkins-instance```
- SSH access to Compute Engine instance 
- Access to GKE API for Jenkins master VM - create a [service account](https://cloud.google.com/sdk/gcloud/reference/auth/activate-service-account) on GKE having role container/Admin
  - [Authenticating services](https://cloud.google.com/kubernetes-engine/docs/how-to/api-server-authentication)
  - Create the Service Account ```gcloud iam service-accounts create ${SERVICE_ACCOUNT_NAME} --display-name="Jenkins Service Account"```
