# Bedrock Heavy Machinery Assistant

A retrieval-augmented chat application built on Amazon Bedrock. It answers questions about heavy machinery (excavators, bulldozers, cranes, forklifts, dump trucks) by grounding an LLM's responses in a Bedrock Knowledge Base backed by an Aurora Serverless PostgreSQL vector store.

> Built as part of the **AWS Future AI Engineer** course.

## How it works

1. Equipment spec sheets (PDFs) are uploaded to S3 and indexed into a Bedrock Knowledge Base, which stores embeddings in Aurora Serverless (pgvector).
2. A Streamlit app takes a user question, classifies it (in/out of scope, safe/unsafe) with a Claude model on Bedrock, retrieves relevant chunks from the knowledge base, and generates a grounded answer.

## Tech stack

- **Infrastructure**: Terraform (VPC, Aurora Serverless v2, S3, Bedrock Knowledge Base, IAM)
- **AI**: Amazon Bedrock (Claude 3 Haiku / 3.5 Sonnet), Bedrock Knowledge Bases
- **Database**: Aurora Serverless PostgreSQL with `pgvector`
- **App**: Python, Streamlit, boto3

## Project structure

```
├── app.py                  # Streamlit chat UI
├── bedrock_utils.py        # Bedrock model + knowledge base calls
├── stack1/                 # Terraform: VPC, Aurora Serverless, S3 bucket
├── stack2/                 # Terraform: Bedrock Knowledge Base
├── modules/
│   ├── database/            # Aurora Serverless module
│   └── bedrock_kb/          # Bedrock Knowledge Base module
├── scripts/
│   ├── aurora_sql.sql       # Enables pgvector and creates the vector table
│   ├── upload_s3.py         # Uploads spec sheets to S3
│   └── spec-sheets/         # Sample equipment spec sheet PDFs
└── requirements.txt
```

## Setup

**Prerequisites**: AWS CLI configured, Terraform >= 0.12, Python >= 3.10

1. **Deploy the data layer** (`stack1/`) — VPC, Aurora Serverless cluster, S3 bucket:
   ```bash
   cd stack1
   terraform init
   terraform apply
   ```
2. **Prepare the database** — run `scripts/aurora_sql.sql` against the new cluster (e.g. via the RDS Query Editor) to enable `pgvector` and create the vector table.
3. **Deploy the knowledge base** (`stack2/`) — using the outputs from step 1 (Aurora ARN, endpoint, secret ARN, S3 bucket ARN):
   ```bash
   cd stack2
   terraform init
   terraform apply
   ```
4. **Upload documents**: set your bucket name in `scripts/upload_s3.py`, then run it to push PDFs from `scripts/spec-sheets/` to S3. Sync the data source in the Bedrock console afterwards so it's indexed.
5. **Run the app**:
   ```bash
   pip install -r requirements.txt
   streamlit run app.py
   ```
   Enter your Knowledge Base ID in the sidebar and start chatting.

## Notes

- No credentials are hardcoded — AWS access is picked up from your local AWS CLI configuration via `boto3`.
- `terraform.tfstate` and `.terraform/` are intentionally excluded; each deployer generates their own.
