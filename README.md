# Intelligent Document Querying System (Heavy Machinery Domain)

An enterprise-grade, Retrieval-Augmented Generation (RAG) system engineered to securely ingest, index, and query technical document corpora using natural language interfaces. This solution leverages Amazon Bedrock for foundational model orchestration, Amazon Aurora Postgres Serverless (with pgvector) for scalable vector storage, and Amazon S3 for secure document housing. The client layer is powered by a Streamlit interactive dashboard, and the entire ecosystem is provisioned deterministically using Terraform for Infrastructure as Code (IaC).

---

## 🏗️ System Architecture

The architecture relies on a multi-stack decoupled structure to isolate database provisioning from the AI orchestration layers, maximizing security, modularity, and resource independence.

### Component Breakdown
*   **Data Layer & Storage:** Amazon S3 serves as the landing zone for technical documentation and specification sheets.
*   **Vector Store & Indexing:** Amazon Aurora Serverless acts as the operational vector database, managing document chunks and high-dimensional semantic vectors using the PostgreSQL `pgvector` extension.
*   **Orchestration Layer:** Amazon Bedrock manages runtime embedding generation and semantic search matching against the database.
*   **Security Guardrails:** A classification model acts as an application firewall, enforcing prompt input boundaries so only domain-relevant inputs execute downstream.
*   **Application Interface:** A Streamlit single-page application built with active session state memory tracks multi-turn conversations and manipulates generation metrics.

---

## ⚙️ Core Ingestion & LLM Processing Pipeline

### 1. Multi-Stage Ingestion Pipeline

The document ingestion process moves through four distinct stages:

1. **Staging:** Source domain PDFs are placed in the local directory structure.
2. **S3 Transmission:** An automated batch script copies files to an S3 bucket while preserving the nested folder hierarchy paths.
3. **Chunking & Embedding:** Syncing the Bedrock Knowledge Base breaks long technical documents into discrete chunks, passes them through an embedding model, and extracts geometric representations.
4. **Vector Upsert:** High-dimensional embeddings are indexed directly inside the target tables in the Aurora Postgres cluster.

### 2. Guardrails & Sampling Mechanics

To protect system resources and guarantee domain safety, input filtering is implemented alongside exact token control:

* **Prompt Guardrail (Input Validation):** Every user prompt runs through an internal classification function (`valid_prompt`). If a prompt falls under an out-of-domain query (e.g., general knowledge, architecture info, profanity, or system prompt injection attempts), execution halts immediately before triggering vector storage retrieval or charging LLM context costs.
* **Temperature:** Adjusts the probability distribution of the next word to control the randomness of the output. Low levels make the model deterministic, predictable, and factual.
* **Top_p (Nucleus Sampling):** Controls the vocabulary size from which the model selects its next token. Low Top_p configurations restrict the pool to a highly probable subset, producing focused and coherent responses.

---

## 📂 Project Directory Structure
project-root/
│
├── stack1
|   ├── main.tf
|   ├── outputs.tf
|   └── variables.tf
|
├── stack2
|   ├── main.tf
|   ├── outputs.tf
|   └── variables.tf
|
├── modules/
│   ├── aurora_serverless/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── bedrock_kb/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── scripts/
│   ├── aurora_sql.sql
│   └── upload_to_s3.py
│
├── spec-sheets/
│   └── machine_files.pdf
│
└── README.md

---

## Step-by-Step Deployment Guide

### Prerequisites
* Fully updated AWS CLI instance with administrative privileges.
* Terraform CLI installed,
* Python 3.10+ virtual environment (`venv`) activated.

#### Step 1: Base Infrastructure Provisioning (Stack 1)
Initialize and build the core network topology, secure private subnets, security groups, the S3 file drop, and the backend Aurora Serverless PostgreSQL cluster
```bash
cd stack1
terraform init
terraform apply
```
#### Step 2: Vector Database Space Setup
Before orchestrating AI tools, prepare the backend database cluster for vector indexing. Log in to the Amazon RDS Console, navigate to the Query Editor, select the my-aurora-serverless database instance, and execute the configuration steps.
1. Activate the Vector Extension Library:
   SELECT * FROM pg_extension;

2. Verify target tables for Bedrock Knowledge Base mapping:
   SELECT table_schema || '.' || table_name as show_tables 
   FROM information_schema.tables 
   WHERE table_type = 'BASE TABLE' AND table_schema = 'bedrock_integration';

#### Step 3: Bedrock & Knowledge Base Architecture (Stack 2)
Navigate to Stack 2. Update local configurations with the cluster endpoints and secret descriptors captured from Stack 1, then build out the Amazon Bedrock integration workspace.
```bash
cd ../stack2
terraform init
terraform apply
```
#### Step 4: Automated Domain Data Ingestion
1. Place technical heavy machinery manuals and specification assets into the local spec-sheets/ folder.
2. Initialize and resolve application runtime dependencies within the terminal:
```bash
pip install -r requirements.txt
```
3. Update the bucket_name inside scripts/upload_to_s3.py to point to the new bucket, then execute the ingestion script.
```bash
python scripts/upload_to_s3.py
```
4. Data Sync Triggering: Open the AWS Console under Amazon Bedrock -> Knowledge Bases, locate my-bedrock-kb, and click the Sync button on the data source to map the S3 content into the vector tables.

## Running the Application
1. Guardrail Validation and Security Testing
An automated integration test suite verifies that prompt injections and out-of-domain queries are blocked before reaching downstream vector databases or LLM processing queues. Run the validation engine using:
```bash
python test.py
streamlit run app.py
```
This interface enables end-users to select foundational models, alter generation parameters via sliders, and query raw data through natural language.

🤝 Credits
This comprehensive application ecosystem was designed, engineered, and validated utilizing best practices developed during the AWS AI Engineer Nanodegree program.
