
# 🚀 How to Deploy a Retrieval-Augmented Generation (RAG) Application on GKE

Imagine teaching an AI to answer questions about Netflix shows with pinpoint accuracy.  
That’s the power of **Retrieval-Augmented Generation (RAG)**—a technique that boosts large language models (LLMs) by pairing them with a searchable knowledge base.

In this guide, you'll deploy a full RAG application on **Google Kubernetes Engine (GKE)** using:

- 🤖 **Mistral-7B** for LLM responses  
- 🧠 **Cloud SQL + pgvector** for vector search  
- ⚙️ **Ray**, **Terraform**, **Helm**, and **JupyterHub** for orchestration  

By the end, you’ll have a working **chat interface** that answers questions about Netflix shows.

---

## 🧠 Step 1: What is RAG?

**RAG** makes LLMs smarter by retrieving relevant context from a dataset before answering a question.

Instead of retraining the model, RAG:
- Retrieves relevant data (e.g., Netflix show descriptions) from a vector DB
- Feeds it into the LLM along with the user query

**Tech Stack:**
- **Model**: `mistral-7b`
- **Database**: Cloud SQL + `pgvector`
- **Pipeline**: Ray, JupyterHub, chat UI on GKE

---

## 🔧 Step 2: Gather Your Tools

Install the following:

- `kubectl` – Kubernetes CLI  
- `terraform` – Infrastructure as Code  
- `helm` – Kubernetes package manager  
- `gcloud` – Google Cloud SDK  

> Download each from their official site.

---

## 🧭 Step 3: Architecture Overview

```text
+----------------+         +-------------+         +----------------------+
| Jupyter Notebook|  -->   | Ray Cluster |  -->    | Cloud SQL + pgvector |
+----------------+         +-------------+         +----------------------+
       |                                                |
       |                                                v
       |                                   +---------------------------+
       +-------------------------------→   |   Chat UI + Mistral-7B    |
                                           +---------------------------+
```
![image](https://github.com/user-attachments/assets/c2a58bb0-8812-410d-a548-cf6a0ea92e55)

This is a **modular, cloud-native, scalable** setup.

---

## 🏗️ Step 4: Deploy Infrastructure with Terraform

### Clone the Repo:
```bash
git clone https://github.com/GoogleCloudPlatform/ai-on-gke.git
cd ai-on-gke/applications/rag
```

### Configure `workloads.tfvars`:
```hcl
project_id     = "your-gcp-project-id"
location       = "us-central1"
cluster_name   = "rag-cluster"
bucket_name    = "my-rag-bucket-123"
```

### Deploy:
```bash
terraform init
terraform apply --var-file workloads.tfvars
```

> Confirm with `yes`. Takes ~10 minutes.  
> ⚠️ Don’t delete `terraform.tfstate`!

---

## 📦 Step 5: Process Netflix Data

### Connect to GKE:
```bash
export NAMESPACE=rag
export CLUSTER_NAME=rag-cluster
export CLUSTER_LOCATION=us-east4

gcloud container clusters get-credentials $CLUSTER_NAME --location=$CLUSTER_LOCATION
```

### Port-forward JupyterHub:
```bash
kubectl port-forward service/proxy-public -n $NAMESPACE 8081:80 &
```

Then open [http://localhost:8081](http://localhost:8081)  
Login with `admin` and the password from:
```bash
terraform output jupyterhub_password
```

### Load Notebook:
In JupyterHub:  
**File > Open From URL** → Paste:
```
https://raw.githubusercontent.com/GoogleCloudPlatform/ai-on-gke/main/applications/rag/example_notebooks/rag-kaggle-ray-sql-interactive.ipynb
```

### Add Kaggle Credentials:
Upload your `kaggle.json`, then update the first notebook cell with your Kaggle username and API key.

### Run the Notebook:
- Downloads the Netflix dataset
- Converts it to vector embeddings via Ray
- Stores embeddings in `pgvector`

⏱️ Takes ~10 minutes.

---

## 💬 Step 6: Launch the Chat UI

### Port-forward the UI:
```bash
kubectl port-forward service/rag-frontend -n $NAMESPACE 8080:8080 &
```

Open [http://localhost:8080](http://localhost:8080)  
Try:  
> _"What’s a good sci-fi show on Netflix?"_

---

## 🧹 Step 7: Clean Up

To remove everything:
```bash
terraform destroy --var-file workloads.tfvars
```

> Confirm with `yes`.  
> 🔧 If the network isn’t deleted, remove it manually from the Google Cloud Console.

---

## 🛠️ Step 8: Troubleshooting

- **Notebook stuck?**  
  ```bash
  kubectl port-forward -n $NAMESPACE service/ray-cluster-kuberay-head-svc 8265:8265
  ```
  Visit [http://localhost:8265](http://localhost:8265)

- **Model fails to load?**  
  Ensure **L4 GPU quota** is available in your selected region.

- **Hugging Face model access error?**  
  Add your Hugging Face token as a Kubernetes secret. See the [GitHub guide](https://github.com/GoogleCloudPlatform/ai-on-gke).

---

## 🎯 Why This Project Matters

This project combines:

- **Infrastructure-as-Code** (Terraform)
- **Scalable processing** (Ray on GKE)
- **Vector search + LLM inference** (pgvector + Mistral)

🔁 Easily swap the Netflix dataset for **internal documents**, **product manuals**, or **customer support data**.

---

## 🙋‍♂️ Let’s Talk!

Have you built something similar with RAG or GKE?  
Feel free to share your thoughts, open an issue, or fork this project!
