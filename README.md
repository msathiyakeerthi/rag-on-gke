---

How to Deploy a Retrieval-Augmented Generation (RAG) Application on GKE: A Step-by-Step Guide
Imagine teaching an AI to answer questions about Netflix shows with pinpoint accuracy. That's the power of Retrieval-Augmented Generation (RAG) - a technique that supercharges large language models (LLMs) by pairing them with a searchable knowledge base.
In this article, I'll walk you through deploying a RAG application on Google Kubernetes Engine (GKE) - step-by-step, like a classroom lab. By the end, you'll have a working chat interface powered by mistral-7b and a Netflix dataset.
Let's get started!
RAG makes LLMs smarter by giving them real-time context from a dataset. Instead of retraining the model (which is costly and slow), RAG uses a vector database to retrieve relevant info - like Netflix show descriptions - and feeds it to the LLM.
Why it matters: It's perfect for domain-specific knowledge or fast-changing content.
Our setup:
AI Model: mistral-7b for answering questions
Vector Database: Cloud SQL with pgvector for storing embeddings.
Tools: A Ray cluster, Jupyter notebook, and chat UI - all on GKE.

Step 1: Gather Your Tools
Before we begin, install the essentials:
kubectl: Control Kubernetes clusters
Terraform: Automate infrastructure setup
Helm: Manage Kubernetes apps
gcloud: Work with Google Cloud

Step 2: Understand the Architecture
Here's how everything fits together:
A Jupyter notebook processes the Netflix dataset from Kaggle.
A Ray cluster turns data into vector embeddings and stores them in pgvector.
A chat UI retrieves context, adds it to your question, and sends it to mistral-7b.

Step 3: Provision Infrastructure with Terraform
Terraform is our blueprint for spinning up GKE, databases, and networking.
Get the Code:
git clone https://github.com/GoogleCloudPlatform/ai-on-gke.git
cd ai-on-gke/applications/rag
Configure:
Edit workloads.tfvars and set:
project_id = "<your-gcp-project-id>"
location = "us-central1"
cluster_name = "rag-cluster"
bucket_name = "my-rag-bucket-123"

Deploy:
terraform init
terraform apply --var-file workloads.tfvars
Confirm with yes. Setup takes ~10 minutes. Don't delete terraform.tfstate-it tracks infrastructure.
Step 4: Process Netflix Data
Connect to GKE:
export NAMESPACE=rag
export CLUSTER_NAME=rag-cluster
export CLUSTER_LOCATION=us-east4
gcloud container clusters get-credentials $CLUSTER_NAME --location=$CLUSTER_LOCATION
Launch JupyterHub:
kubectl port-forward service/proxy-public -n $NAMESPACE 8081:80 &
Open http://localhost:8081.
 Login using admin and the password from the terraform output jupyterhub_password.
Load Notebook:
In JupyterHub:
 File > Open From URL, then paste:
https://raw.githubusercontent.com/GoogleCloudPlatform/ai-on-gke/main/applications/rag/example_notebooks/rag-kaggle-ray-sql-interactive.ipynb
Add Kaggle API Key:
Upload kaggle.json, and update the first notebook cell with your username and key.
Run All Cells:
This downloads the Netflix dataset, generates embeddings with Ray, and saves them to Cloud SQL.
Expect ~10 minutes runtime.
Step 5: Launch the Chat UI
kubectl port-forward service/rag-frontend -n $NAMESPACE 8080:8080 &
Then, go to http://localhost:8080
 Ask: "What's a good sci-fi show on Netflix?" and watch the magic unfold.
Step 6: Clean Up
To avoid cloud charges:
terraform destroy --var-file workloads.tfvars
Confirm with yes. If the VPC network sticks around, delete it manually via GCP Console.

---

Step 7: Troubleshooting Tips
Notebook stuck?

kubectl port-forward -n $NAMESPACE service/ray-cluster-kuberay-head-svc 8265:8265
Then open http://localhost:8265 to debug Ray.
Model not loading?
 Make sure you have L4 GPU quota in your region.
HF gated model?
 Add your Hugging Face access token as a Kubernetes secret (details in the GitHub tutorial).

---

Why This Project Matters
This project showcases a modern AI deployment stack:
Infrastructure-as-Code with Terraform
Data preprocessing and embedding with Ray
Real-time inference with Mistral + pgvector
Scalability with GKE

You can easily swap the Netflix dataset for your own - internal docs, product catalogs, or support tickets - and adapt this to enterprise needs.
Let's Chat
Have you explored RAG or GKE for AI workloads?
 Drop your thoughts or questions in the comments. I'd love to hear how you're using these tools!
