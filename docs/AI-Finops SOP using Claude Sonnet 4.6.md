**AI-Powered FinOps SaaS --- Technical SOP & Implementation Guide**

**1️⃣ Network & Environment Setup (VPC & Subnets)**

**AWS Services:**

- VPC 🟦

- Subnets 🟩

- Internet Gateway 🌐

- NAT Gateway 🟧

**Steps in AWS Console:**

1.  **Create VPC**

    - VPC → Create VPC

    - CIDR: 10.0.0.0/16

    - Name: finops-prod-vpc

2.  **Create Subnets**

  ----------------------------------------------------
  **Type**   **CIDR**      **Purpose**        **AWS
                                              Icon**
  ---------- ------------- ------------------ --------
  Public     10.0.1.0/24   ALB                🟩

  Public     10.0.2.0/24   ALB                🟩

  Private    10.0.3.0/24   ECS + Aurora       🟩
                           Serverless         

  Private    10.0.4.0/24   ECS + Aurora       🟩
                           Serverless         
  ----------------------------------------------------

3.  **Create Internet Gateway**

    - IGW → Attach to VPC

    - Public route table → 0.0.0.0/0 → IGW 🌐

4.  **Create NAT Gateway**

    - Allocate Elastic IP → Create NAT in public subnet

    - Private route table → 0.0.0.0/0 → NAT Gateway 🟧

**2️⃣ Database Layer --- Aurora Serverless v2 + RDS Proxy**

**AWS Services:**

- Aurora Serverless v2 🟪

- RDS Proxy 🟫

**Steps:**

1.  **Create Aurora Serverless v2**

    - RDS → Create Database → Engine: Aurora PostgreSQL

    - Capacity: Serverless v2

    - Private subnets only, no public access

    - Enable encryption, automated backups, performance insights

    - Store credentials in **Secrets Manager**

2.  **Create RDS Proxy**

    - RDS → Proxies → Create → Target: Aurora cluster

    - Enable IAM authentication

    - Secrets Manager credentials

    - Place in private subnets

    - Benefits: Connection pooling for ECS tasks

**3️⃣ AI Layer --- Claude Sonnet 4.6**

**AWS Services:**

- Amazon Bedrock 🟨

**Steps:**

- Bedrock → Model Access → Request Claude Sonnet 4.6 → Wait for approval

**4️⃣ FinOps SaaS Application --- FastAPI & Docker**

**4.1 Project Structure**

finops-app/

│

├── Dockerfile

├── requirements.txt

└── app/

├── main.py

├── routes.py

├── database.py

├── models.py

├── ai_service.py

└── cost_service.py

**4.2 Code Snippets**

**main.py**

from fastapi import FastAPI

from app.routes import router

app = FastAPI(title=\"AI-Powered FinOps SaaS\")

app.include_router(router)

\@app.get(\"/\")

def root():

return {\"message\": \"Hello, FinOps SaaS!\"}

**routes.py**

from fastapi import APIRouter

from app.cost_service import fetch_cost_data

from app.ai_service import analyze_costs

router = APIRouter()

\@router.get(\"/costs\")

def get_costs():

data = fetch_cost_data()

return data

\@router.get(\"/ai\")

def get_ai_recommendations():

costs = fetch_cost_data()

recommendations = analyze_costs(costs)

return recommendations

**database.py**

import os

from sqlalchemy import create_engine

DB_HOST = os.getenv(\"DB_HOST\", \"localhost\")

DB_NAME = os.getenv(\"DB_NAME\", \"finopsdb\")

DB_USER = os.getenv(\"DB_USER\", \"admin\")

DB_PASSWORD = os.getenv(\"DB_PASSWORD\", \"password\")

DATABASE_URL =
f\"postgresql+psycopg2://{DB_USER}:{DB_PASSWORD}@{DB_HOST}/{DB_NAME}\"

engine = create_engine(DATABASE_URL)

**models.py**

from sqlalchemy import Column, Integer, String

from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class CostRecord(Base):

\_\_tablename\_\_ = \"cost_records\"

id = Column(Integer, primary_key=True)

service = Column(String)

cost = Column(Integer)

month = Column(String)

**ai_service.py**

def analyze_costs(cost_data):

\# Placeholder for Claude Sonnet integration

return {\"recommendation\": \"Reduce unused EC2 instances\"}

**cost_service.py**

def fetch_cost_data():

\# Placeholder for AWS Cost Explorer integration

return {\"total_cost\": 12345, \"services\": \[\"EC2\", \"RDS\",
\"S3\"\]}

**4.3 Dockerfile**

FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app/ app/

EXPOSE 8000

CMD \[\"uvicorn\", \"app.main:app\", \"\--host\", \"0.0.0.0\",
\"\--port\", \"8000\"\]

**requirements.txt**

fastapi

uvicorn

sqlalchemy

psycopg2-binary

boto3

**5️⃣ Docker Local Build & Test**

1.  **Build Docker Image**

cd C:\\Users\\bodde\\.docker\\finops-app

docker build -t finops-api .

2.  **Run Locally**

docker run -it -p 8000:8000 \--name finops-api finops-api:latest

- Test endpoint: http://localhost:8000/

- Expected response: {\"message\": \"Hello, FinOps SaaS!\"}

3.  **Stop & Remove Container**

docker stop finops-api

docker rm finops-api

**6️⃣ Push to AWS ECR**

1.  **Create ECR Repository**

    - AWS Console → ECR → Create Repository → finops-api

2.  **Tag Image**

docker tag finops-api:latest
\<aws_account_id\>.dkr.ecr.\<region\>.amazonaws.com/finops-api:latest

3.  **Login & Push**

aws ecr get-login-password \--region \<region\> \| docker login
\--username AWS \--password-stdin
\<aws_account_id\>.dkr.ecr.\<region\>.amazonaws.com

docker push
\<aws_account_id\>.dkr.ecr.\<region\>.amazonaws.com/finops-api:latest

**7️⃣ ECS Fargate Deployment**

1.  **Task Definition**

    - Launch type: Fargate

    - Container image: \<ECR URI\>

    - Port: 8000

    - Env variables: DB_HOST, DB_NAME, BEDROCK_MODEL

    - IAM Role: access Secrets Manager, RDS Proxy, Bedrock, Cost
      Explorer

2.  **ECS Cluster**

    - Fargate → Private subnets → Optional auto scaling

3.  **ECS Service**

    - Attach ALB → HTTP 80 → Target group

    - Desired tasks: 2 → Auto Scaling CPU 50%

**8️⃣ Optional Next Steps (AWS Console Implementation)**

1.  **CloudWatch Logs / Metrics** → ECS Task Definition → Logging →
    CloudWatch

2.  **HTTPS on ALB** → ACM Certificate → Attach to ALB → Listener HTTPS
    443

3.  **AWS WAF** → Web ACL → Attach ALB → SQLi/XSS rules + rate limits

4.  **Secrets Manager** → Store DB/API credentials → Inject into ECS
    environment variables

5.  **Multi-Tenant Cognito** → Create User Pool → App Client → Map roles
    → ALB integration

6.  **Auto-Scaling**

    - ECS: CPU target 50%

    - Aurora: Serverless min/max ACU, auto-pause

7.  **X-Ray Distributed Tracing** → ECS task IAM role + SDK

8.  **CI/CD** → CodePipeline or GitHub Actions → Build Docker → Push ECR
    → Deploy ECS
