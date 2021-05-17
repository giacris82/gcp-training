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
  
  ```gcloud init```
- Please choose `europe-west3-b`as default zone and `europe-west3` as default region- Install `kubectl`.

    ```gcloud componets install kubectl```
- Configure project: 
    
    `gcloud config set project <YOUR_GCP_PROJECT>`
### Install GKE

```
PROJECT_ID=training-project-313415
gcloud config set project $PROJECT_ID
PROJECT_ID=$(gcloud config get-value project)
//Enable Kubernetes Engine API
gcloud services enable compute.googleapis.com \
container.googleapis.com \
servicemanagement.googleapis.com \
cloudresourcemanager.googleapis.com \
    --project $PROJECT_ID

// Create aK8s Cluster
K8S_CLUSTERNAME=gltraining-k8s 
gcloud container clusters create $K8S_CLUSTERNAME --machine-type=n1-standard-2 --num-nodes=3 --zone=europe-west3-b --project=$PROJECT_ID
//set the kubeconfig
gcloud container clusters get-credentials $K8S_CLUSTERNAME--zone=europe-west3-b --project=$PROJECT_ID
```

### Create Jenkins instance- Create Compute Engine instance
```gcloud beta compute --project=$PROJECT_ID instances create jenkins --zone=europe-west3-b --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --tags=jenkins-instance --image=debian-10-buster-v20210316 --image-project=debian-cloud --boot-disk-size=20GB --boot-disk-type=pd-balanced --boot-disk-device-name=jenkins-boot --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any```

- Grant access to Jenkins UI
```MY_IP=5.90.105.199
gcloud compute firewall-rules --project=$PROJECT_ID create fw-jenkins-http --source-ranges=$MY_IP --allow=tcp:443,tcp:8080 --direction=IN --network=default --target-tags=jenkins-instance
```
- Install Java and Jenkins on the VM
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
```


- SSH access to Compute Engine instance

```gcloud beta compute ssh --zone "europe-west3-b" "jenkins" --project "training-project-313415"```

- Access to GKE API for Jenkins master VM - create a [service account](https://cloud.google.com/sdk/gcloud/reference/auth/activate-service-account) on GKE having role container/Admin
  - [Authenticating services](https://cloud.google.com/kubernetes-engine/docs/how-to/api-server-authentication)
  - Create the Service Account ```gcloud iam service-accounts create ${SERVICE_ACCOUNT_NAME} --display-name="Jenkins Service Account"```

  - Assign the required roles to the service account:  ```export PROJECT=$(gcloud info --format='value(config.project)')
export SA_EMAIL=$(gcloud iam service-accounts list --filter="name:jenkins-gce" \
 --format='value(email)')
gcloud projects add-iam-policy-binding --member serviceAccount:$SA_EMAIL \
 --role roles/container.admin $PROJECT
gcloud projects add-iam-policy-binding --member serviceAccount:$SA_EMAIL \
 --role roles/container.clusterAdmin $PROJECT
gcloud projects add-iam-policy-binding --member serviceAccount:$SA_EMAIL \
 --role roles/iam.serviceAccountUser $PROJECT```

  - Verify roles: ```gcloud projects get-iam-policy $PROJECT```

- Grab the JSON service account key: 
```gcloud iam service-accounts keys create --iam-account $SA_EMAIL jenkins-gce.json```
