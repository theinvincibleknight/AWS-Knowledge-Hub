# 🚀 Step‑by‑Step Guide: Deploying an ECS Service with AWS Copilot

AWS Copilot is a CLI tool that simplifies deploying applications to Amazon ECS. This guide walks you through deploying a sample service into ECS using Copilot.

---

## 📋 Prerequisites

Before you begin, ensure you have:

1. **AWS CLI installed and configured**  
   ```bash
   aws configure
   ```
   Provide your AWS access key, secret key, region, and output format.

2. **Install AWS Copilot CLI (Ubuntu example)**  
   ```bash
   curl -Lo copilot https://github.com/aws/copilot-cli/releases/latest/download/copilot-linux \
   && chmod +x copilot \
   && sudo mv copilot /usr/local/bin/copilot
   ```
   Verify installation:
   ```bash
   copilot --help
   ```

3. **VPC setup**  
   - You need an existing **VPC** with at least **two private subnets**.  
   - In this guide, we use private subnets only because we’re deploying a **backend service type** (no public internet exposure).

4. **ACM Certificate**  
   - Request or import an ACM certificate for your domain in the same region.  
   - This certificate will be referenced in your Copilot manifest file to enable HTTPS and provide a URL after deployment.

---

## 1. Clone the Sample Application
```bash
git clone https://github.com/aws-samples/aws-copilot-sample-service example-app
cd example-app
```

---

## 2. Initialize the Copilot Application
```bash
copilot app init
```
Provide an application name (e.g., `dashboard`).

---

## 3. Initialize an Environment
```bash
copilot env init \
  --name uat \
  --app dashboard \
  --import-vpc-id vpc-02452dce7b9b446e1 \
  --import-private-subnets subnet-07d1a098cc25e5b0b,subnet-055300459009ccc40 \
  --import-cert-arns arn:aws:acm:ap-south-1:891398765521:certificate/07645bcb-338e-4b83-bd37-fc3886750403
```

---

## 4. Deploy the Environment
```bash
copilot env deploy --name uat
```

---

## 5. Initialize the Service
```bash
copilot svc init
```
Provide:
- Service name (e.g., `example-app`)
- Service type (e.g., **Backend Service** for private workloads)
- Dockerfile location

---

## 6. Deploy the Service
```bash
copilot svc deploy --name example-app --env uat
```

Copilot will:
- Build and push your Docker image to ECR.
- Create ECS task definitions and services.
- Attach load balancers and networking.
- Deploy the service into your environment.

---

## 7. Verify Deployment
After deployment, Copilot will output the service URL (using your ACM certificate). Open it in your browser to confirm the service is running.

---

## 📊 Summary of Steps
1. Install AWS CLI and Copilot CLI  
2. Ensure VPC, private subnets, and ACM certificate exist  
3. Clone sample app → `git clone …`  
4. Initialize Copilot app → `copilot app init`  
5. Initialize environment → `copilot env init …`  
6. Deploy environment → `copilot env deploy …`  
7. Initialize service → `copilot svc init`  
8. Deploy service → `copilot svc deploy …`  
9. Verify service URL  

---

For more details, see the official documentation: [AWS Copilot CLI](https://aws.github.io/copilot-cli/)

---

## 📑 Customizing the Manifest File

After initializing and deploying your service with Copilot, you’ll find a manifest file created at:

```
copilot/<service-name>/manifest.yml
```

This file defines how your ECS service runs — including networking, scaling, environment overrides, secrets, and more. You can edit it to customize your deployment.

---

### Example Manifest File

Here’s a sample manifest for a backend service called **dashboard**:

```yaml
# The manifest for the "dashboard" service.
name: dashboard
type: Backend Service

# Routing configuration
http:
  path: 'api/dashboard'
  healthcheck: '/api/dashboard/v1/health'
  alias: '${COPILOT_ENVIRONMENT_NAME}.example.com'

network:
  vpc:
    placement: private   # Use private subnets only

# Container configuration
image:
  build: Dockerfile
  port: 8081

variables:
  DW_CONNECTION_NAME: TESTDW

cpu: 1024
memory: 2048
count: 1
exec: true

# Secrets stored in SSM Parameter Store
secrets:
  DB_HOST: /copilot/${COPILOT_APPLICATION_NAME}/${COPILOT_ENVIRONMENT_NAME}/secrets/DB_HOST
  DB_USER: /copilot/${COPILOT_APPLICATION_NAME}/${COPILOT_ENVIRONMENT_NAME}/secrets/DB_USER
  DB_PASSWORD: /copilot/${COPILOT_APPLICATION_NAME}/${COPILOT_ENVIRONMENT_NAME}/secrets/DB_PASSWORD

# Environment-specific overrides
environments:
  uat:
    http:
      alias: 'uat.example.com'
    count:
      range:
        min: 1
        max: 2
      cooldown:
        in: 120s
        out: 60s
      cpu_percentage: 70
      memory_percentage: 80
      requests: 1000
    variables:
      DB_SCHEMA: DASH_SYS_CONFIG
      OTP_SERVICE_BASE_PATH: 'http://otp-service.uat.local:3000/api/otp'

  prod:
    http:
      alias: 'prod.example.com'
    cpu: 2048
    memory: 4096
    count:
      range:
        min: 1
        max: 4
      cooldown:
        in: 120s
        out: 60s
      cpu_percentage: 70
      memory_percentage: 80
      requests: 1000
    variables:
      DB_SCHEMA: DASH_SYS_CONFIG
      AUTH_CLIENT_ID: 6dcfa539-227d-4fbb-8713-b20bdvacc9a6
```

---

### 🔎 Key Sections Explained

- **name / type**: Defines the service name and type (`Backend Service` for private workloads).
- **http**: Configures routing, health checks, and domain aliases (using ACM certificates).
- **network**: Specifies subnet placement (`private` for backend services).
- **image**: Defines Docker build context and container port.
- **variables**: Environment variables available inside the container.
- **cpu / memory / count**: Resource allocation and number of tasks.
- **exec**: Enables `copilot svc exec` to run commands inside the container.
- **secrets**: Secure values stored in AWS SSM Parameter Store.
- **environments**: Overrides for specific environments (`uat`, `prod`), including scaling ranges, domain aliases, and environment-specific variables.

---

## 🛠️ Workflow After Customization

1. Edit the manifest file to match your service requirements.
2. Save changes.
3. Redeploy the service:
   ```bash
   copilot svc deploy --name dashboard --env uat
   ```
4. Copilot will update ECS with the new configuration.

---

## 🔎 Detailed Understanding of AWS Copilot Deployment Process

AWS Copilot abstracts away a lot of complexity, but under the hood it orchestrates **CloudFormation stacks, IAM roles, ECS resources, ECR repositories, and networking**. Here’s the full breakdown:

---

## 1. `copilot app init`
- **Purpose**: Creates a new Copilot application (logical grouping of services and environments).
- **Behind the scenes**:
  - A **CloudFormation stack** is created in your AWS account.
  - This stack provisions:
    - IAM roles and policies for Copilot to manage resources.
    - Metadata storage (SSM parameters) to track app configuration.
  - No ECS service or environment is created yet — this is just the foundation.

---

## 2. `copilot env init`
- **Purpose**: Defines an environment (e.g., `uat`, `prod`) where services will run.
- **Behind the scenes**:
  - A new **CloudFormation stack** is prepared for the environment.
  - Resources created:
    - VPC configuration (or imported VPC/subnets if specified).
    - Security groups.
    - Load balancers (ALB/NLB depending on service type).
    - ACM certificate association for HTTPS.
    - Environment metadata stored in SSM.
  - Still no ECS service yet — this sets up the infrastructure.

---

## 3. `copilot env deploy`
- **Purpose**: Deploys the environment stack.
- **Behind the scenes**:
  - CloudFormation provisions the environment resources defined above.
  - Networking, routing, and certificates are now ready.
  - ECS cluster is created for the environment.
  - IAM roles for ECS tasks and services are provisioned.

---

## 4. `copilot svc init`
- **Purpose**: Defines a new service (your containerized workload).
- **Behind the scenes**:
  - Copilot generates a **manifest file** (`copilot/<service-name>/manifest.yml`).
  - Creates an **ECR repository** for storing Docker images.
  - No ECS service is created yet — this step only sets up definitions and storage.

---

## 5. `copilot svc deploy`
- **Purpose**: Deploys the service into the chosen environment.
- **Behind the scenes**:
  1. **Builds Docker image** using your project’s `Dockerfile`.
  2. **Pushes image to ECR** (repository created during `svc init`).
  3. **CloudFormation stack for the service** is created:
     - ECS Task Definition (CPU, memory, ports, environment variables, secrets).
     - ECS Service (desired count, scaling policies).
     - Load Balancer rules (path-based routing, target groups).
     - Security groups and IAM roles for the service.
  4. **Service URL generated**:
     - Based on the manifest file’s `http.alias` and ACM certificate.
     - Copilot outputs the URL after deployment.

---

## 6. After Deployment
- ECS tasks are running in your environment.
- Load balancer routes traffic to your service.
- You can access the service via the URL Copilot provides.
- Scaling, secrets, and overrides are managed via the manifest file.

---

## 📊 Visualization Diagram

Here’s a conceptual visualization of the workflow:

```
+-------------------+
| copilot app init  |
+-------------------+
        |
        v
 Creates App Stack (IAM roles, metadata)

+-------------------+
| copilot env init  |
+-------------------+
        |
        v
 Prepares Env Stack (VPC, subnets, SGs, ALB, ACM)

+-------------------+
| copilot env deploy|
+-------------------+
        |
        v
 Deploys Env Stack (ECS cluster, networking ready)

+-------------------+
| copilot svc init  |
+-------------------+
        |
        v
 Creates Service Definition (manifest.yml, ECR repo)

+-------------------+
| copilot svc deploy|
+-------------------+
        |
        v
 Builds Docker image → Pushes to ECR
 Creates Service Stack:
   - ECS Task Definition
   - ECS Service
   - Load Balancer rules
   - Target groups
   - IAM roles
   - Security groups
 Provides Service URL (from manifest + ACM)
```

---
