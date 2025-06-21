# GKE Auto Deployment via Azure DevOps Pipeline
This document explains the CI/CD integration between **Azure Repos** and **Google Kubernetes Engine (GKE)** using a **bastion server** for secure cluster access.
## 🔧 Architecture Overview
Azure Repos --> Azure Pipeline --> Build + Push Docker Image --> Bastion Server --> GKE Deployment Rollout
# Azure Reposistory Agent Intgration:
RIL-DevOps -->Jashwanth-DevOps -->Settings --> Agent pools --> Add Pool --> Self hosted --> create pool --> Name: Jhapcore
# Now create agent inside the nodepool
Jhapcore --> New Agent --> Get the agent 
# Install the System prerequisites in the VM which we neeed to integrate to Azure Repo.
### Steps:
1. **Download the agent**
    - https://download.agent.dev.azure.com/agent/4.258.1/vsts-agent-linux-x64-4.258.1.tar.gz
2. **Create the agent**
   -  ~/$ mkdir  myagent && cd myagent
   - ~/myagent$  tar zxvf ~/Downloads/vsts-agent-linux-x64-4.258.1.tar.gz
3. **Configure the agent**
   - ~/myagent$  ./config.sh
4. **Optionally run the agent interactively**
   - ~/myagent$ ./run.sh

## 🚀 CI/CD Pipeline Flow
### Trigger:
- Any commit or merge to `main` branch triggers the pipeline.
### Stages:

1. **Build**
   - Compiles the Java application (Maven/Gradle)
   - Generates a JAR file
2. **Docker Image**
   - Builds a Docker image from the JAR
   - Tags it as `asia-south1-docker.pkg.dev/<project>/<repo>/sample-project:<build-id>`

3. **Push to Google Container Registry (GCR)**
   - Authenticates using a **GCP Service Account JSON key**
   - Pushes the image to GCR

4. **Deploy to GKE (via Bastion Server)**
   - SSH into the bastion host from Azure Pipeline
   - From the bastion, run `kubectl` commands to:
     - Update the image in the deployment
     - Restart the pod using `kubectl rollout restart`

---

## 🔐 gcp-service-account-permissions
- Push Docker images to Google Container Registry (GCR)
- Deploy and manage applications on Google Kubernetes Engine (GKE)

## 🧾 Required IAM Roles

| Purpose                           | Role Name                       | Role ID                          | Description                                               |
|----------------------------------|----------------------------------|----------------------------------|-----------------------------------------------------------|
| Push Docker images to GCR        | Storage Admin                    | `roles/storage.admin`            | Full control over Cloud Storage buckets (includes GCR)   |
| Deploy workloads to GKE          | Kubernetes Engine Developer      | `roles/container.developer`      | Deploy, update, and manage GKE resources                  |
| View GKE cluster details         | Kubernetes Engine Cluster Viewer | `roles/container.clusterViewer`  | View GKE cluster configurations (needed for `kubectl`)   |
| (Optional) Run as service account| Service Account User             | `roles/iam.serviceAccountUser`   | Required for some tools to impersonate the service account |

## 🔗 Azure DevOps Service Connection with GCP Service Account
### 1. Generate GCP Service Account Key
-  create a key in JSON format in the service accunt created in GCP IAM
-  This JSON will be used to authenticate Azure DevOps to GCP services (like GCR & GKE).

### 2. Create Service Connection in Azure DevOps
- Go to **Project Settings → Service Connections → New service connection**.
- Choose **Google Cloud**.
- Upload the **Service Account JSON key**.
- Choose **GCP Project ID** and give a name (e.g., `gcp-cicd-connection`).
---

## 🌐 Infrastructure Details

- **GKE Cluster**: `demo-gke`
- **Region**: `asia-south1`
- **Namespace**: `jhap-core`
- **Deployment Name**: `sample-app`
- **GCR Repo**: `asia-south1-docker.pkg.dev/lyrical-chassis-459113-c9/sample-project`

---

## 📜 GKE Commands (via Bastion)

```bash
- kubectl set image deployment/sample-app sample-app=asia-south1-docker.pkg.dev/lyrical-chassis-459113-c9/sample-project:<build-id> -n jhap-core
- kubectl rollout status deployment/sample-app -n jhap-core
- gcloud container clusters resize demo-gke   --node-pool slim-pool   --num-nodes 3  --region asia-south
- kubectl edit deploy <deployment name>  -n <namespace> -o yaml
- kubectl scale deployment <deployment-name> --replicas=<count> -n <namespace>
- 
