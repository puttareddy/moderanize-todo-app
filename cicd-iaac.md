# CI/CD & Infrastructure as Code

## CI/CD Pipeline (GitHub Actions)

### Overview

The CI/CD pipeline implements a four-stage continuous integration and continuous deployment workflow with a two-approver manual gate before production deployment. The pipeline ensures code quality, validates the FastAPI backend modernization against the Modernization Constitution, enforces security standards, and provides controlled promotion from development to staging to production environments.

### Pipeline Structure

The GitHub Actions workflow consists of four sequential stages:
- Stage 1: Build - Compile, package, and prepare artifacts
- Stage 2: Test - Execute test suite, validation agent, and security scans
- Stage 3: Staging Deploy - Deploy to staging environment automatically on main branch merge
- Stage 4: Production Gate - Manual approval requiring two distinct approvers before production deployment

### Stage 1: Build

The Build stage runs on all pull requests and pushes to prepare deployment artifacts. This stage validates that the code compiles, Docker images build successfully, and Helm charts are valid.

#### Build Steps

- Checkout code from git repository using GitHub actions checkout
- Set up Python 3.11+ environment for FastAPI backend
- Install backend dependencies from backend/requirements.txt (fastapi, uvicorn, pydantic, httpx, python-keycloak, pytest, pytest-asyncio, pytest-cov, black, ruff, mypy)
- Set up Node.js 20+ environment for frontend build (existing frontend preserved)
- Install frontend dependencies from frontend/package.json (react, react-dom, vite, @vitejs/plugin-react)
- Run backend code formatting check using black with --check flag
- Run backend linting using ruff check on backend source code
- Run backend type checking using mypy on backend source code
- Build FastAPI backend Docker image using backend/Dockerfile with multi-stage build process
- Tag Docker image with git commit SHA and branch name
- Build frontend production bundle using npm run build in frontend directory
- Build nginx Docker image using frontend/Dockerfile with production build
- Tag frontend Docker image with git commit SHA and branch name
- Build Keycloak service Docker image if keycloak configuration changed
- Run Helm chart linting using helm lint on terraform/helm/todo-app chart
- Generate build artifacts including Docker image manifests, Helm chart tarball, and build metadata
- Upload build artifacts to GitHub Actions artifacts for download in later stages

#### Build Conditions

- Runs on all pull request events (opened, synchronize, reopened)
- Runs on all push events to any branch
- Fails immediately if code formatting fails (black --check returns non-zero exit)
- Fails immediately if linting fails (ruff check returns non-zero exit)
- Fails immediately if type checking fails (mypy returns non-zero exit)
- Fails immediately if Docker build fails for any service (backend, frontend, Keycloak)
- Fails immediately if frontend build fails (npm run build returns non-zero exit)
- Fails immediately if Helm chart linting fails

#### Build Outputs

- FastAPI backend Docker image pushed to container registry (ECR or GitHub Container Registry)
- Frontend nginx Docker image pushed to container registry
- Keycloak Docker image pulled from quay.io/keycloak/keycloak repository
- Helm chart tarball artifact containing todo-app chart
- Build metadata file containing git SHA, branch, build timestamp, and image tags

---

### Stage 2: Test

The Test stage runs after successful Build stage to validate code quality, run automated tests, execute the Validation Agent, and perform security scanning. This stage ensures all changes comply with the Modernization Constitution and do not introduce regressions.

#### Test Steps

- Download build artifacts from previous stage
- Pull Docker images from container registry
- Set up test environment with Docker Compose or Kubernetes kind cluster
- Start test infrastructure including Keycloak service for authentication testing
- Run backend unit tests using pytest with coverage reporting
- Run backend integration tests using httpx test client against FastAPI endpoints
- Run frontend smoke tests using Playwright or Cypress against React application
- Execute Validation Agent against pull request diff
- Run validation agent with five validation modules: Constitution Compliance, Security and Authentication, Data and Privacy, Performance and Scalability, Accessibility and User Experience
- Generate validation agent report with per-module scores and overall 0-100 score
- Check for CRITICAL violations that block merge
- Run security dependency scan using safety or pip-audit on backend requirements
- Run static code analysis using bandit or semgrep on backend source code
- Run Docker image security scan using trivy or grype on built images
- Generate test coverage report using pytest-cov with HTML output
- Run performance benchmark tests against API endpoints
- Run concurrent request tests to verify thread-safe state management
- Compare response times against Express baseline from original implementation
- Generate test artifacts including test results, coverage reports, validation agent report, and security scan reports

#### Test Conditions

- Runs only after Build stage completes successfully
- Runs on all pull request events
- Runs on all push events to main branch
- Fails immediately if any unit test fails (pytest returns non-zero exit)
- Fails immediately if any integration test fails (httpx tests return non-zero exit)
- Fails immediately if Validation Agent score is below threshold (less than 70)
- Fails immediately if Validation Agent detects any CRITICAL violations
- Fails immediately if security scan detects HIGH or CRITICAL vulnerabilities
- Fails immediately if code coverage is below threshold (less than 80%)
- Fails immediately if performance regression exceeds tolerance (more than 20% slower than baseline)
- Skips frontend smoke tests if frontend code has not changed (using path filters)

#### Test Outputs

- Unit test results with pass/fail status per test case
- Integration test results with API response verification
- Validation Agent report with module scores, overall score, and critical violations
- Security scan report with vulnerability findings and severity levels
- Code coverage report with percentage and uncovered lines
- Performance benchmark results with response times and baseline comparison
- Thread-safety test results with concurrent request verification
- Test artifacts uploaded to GitHub Actions for review

#### Validation Agent Integration

The Validation Agent runs as part of Stage 2 and enforces the Modernization Constitution. The agent loads the constitution from specs/main/constitution.md, analyzes the pull request diff, runs all 50 named checks across five validation modules, and generates a compliance score from 0 to 100.

- **Module 1: Constitution Compliance** - Validates business logic preservation, data integrity, API contract compliance, frontend compatibility, deployment orchestration, and CORS configuration (12 checks, 81 maximum points, 30% weight)
- **Module 2: Security and Authentication** - Validates JWT token implementation, Keycloak integration, authentication flow, authorization foundation, and CORS security (10 checks, 52.5 maximum points, 25% weight)
- **Module 3: Data and Privacy** - Validates in-memory state management, thread-safety patterns, data structure preservation, JSONPlaceholder integration, and soft-delete logic (12 checks, 101 maximum points, 25% weight)
- **Module 4: Performance and Scalability** - Validates async patterns, concurrent request handling, caching efficiency, and memory usage (8 checks, 13 maximum points, 10% weight)
- **Module 5: Accessibility and User Experience** - Validates WCAG 2.1 AA compliance, semantic HTML, keyboard navigation, and error handling (8 checks, 14 maximum points, 10% weight)

**Scoring Thresholds:**
- Overall score of 90-100: Excellent compliance, recommended for merge
- Overall score of 70-89: Good compliance with minor issues, review before merge
- Overall score of 50-69: Marginal compliance with significant issues, requires fixes before merge
- Overall score of 0-49: Poor compliance, must be fixed before merge consideration
- Any CRITICAL check failure: Automatic FAIL status regardless of overall score

**Critical Check List (Auto-Fail):**
- CC-001 through CC-011: All Constitution Compliance CRITICAL checks for business logic and API contracts
- SA-001, SA-002, SA-008: Security CRITICAL checks for JWT validation and authentication blocking
- DP-001 through DP-010: Data and Privacy CRITICAL checks for state management and data integrity

The Validation Agent posts a summary comment to the pull request with overall score, module scores, critical violations, and remediation guidance. CRITICAL violations block the pull request from merging until all failures are addressed and re-validated.

---

### Stage 3: Staging Deploy

The Staging Deploy stage runs automatically after successful Test stage on merges to the main branch. This stage deploys the validated code to the staging environment for final verification before production promotion.

#### Staging Deploy Steps

- Download build artifacts and test artifacts from previous stages
- Pull Docker images from container registry with appropriate tags
- Download Helm chart tarball from build artifacts
- Configure Terraform backend state file pointing to staging AWS account and region
- Initialize Terraform working directory with staging configuration
- Run Terraform plan to preview infrastructure changes in staging environment
- Apply Terraform infrastructure changes using terraform apply with auto-approve
- Wait for Terraform to complete and verify infrastructure provisioning success
- Deploy Helm chart to staging EKS cluster using helm upgrade command
- Apply staging-specific Helm values override file (values-staging.yaml)
- Verify Helm release status using helm status command
- Run post-deployment health checks against staging endpoints
- Verify staging backend health at staging.domain.com/health endpoint
- Verify staging frontend accessibility at staging.domain.com
- Verify Keycloak service health in staging environment
- Run smoke tests against staging API endpoints using staging domain
- Verify JWT token validation works with staging Keycloak instance
- Verify in-memory state management functions correctly in staging
- Generate deployment artifact with Terraform state snapshot, Helm release manifest, and health check results

#### Staging Deploy Conditions

- Runs only after Test stage completes successfully
- Runs automatically on push to main branch
- Skips on pull request events unless manually triggered
- Fails immediately if Terraform plan shows destructive changes without explicit approval
- Fails immediately if Terraform apply fails for any reason (infrastructure provisioning errors)
- Fails immediately if Helm chart deployment fails (Helm upgrade returns non-zero exit)
- Fails immediately if health checks fail against staging endpoints
- Rolls back failed deployment automatically using Helm rollback and Terraform rollback
- Sends notification to team on deployment success or failure

#### Staging Deploy Outputs

- Terraform state file updated with staging infrastructure changes
- Helm release deployed to staging namespace in EKS cluster
- Staging environment fully operational with all services healthy
- Health check results confirming all endpoints responding correctly
- Smoke test results confirming API functionality
- Deployment artifact with infrastructure state for reference

---

### Stage 4: Production Gate

The Production Gate stage provides a manual approval checkpoint requiring two distinct approvers before deploying to production. This stage ensures human review and approval of changes affecting the production environment.

#### Production Gate Steps

- Create GitHub Actions environment for production deployment approval
- Configure environment protection rules requiring two reviewers
- Set up deployment approval workflow with manual trigger
- Display deployment summary from previous Staging Deploy stage
- Show test results summary from Test stage including Validation Agent score
- Show security scan results and vulnerability status
- Show performance benchmark results compared to baseline
- Show Terraform plan for production environment changes
- Request manual approval from two distinct GitHub users
- Verify both approvers have write access to the repository
- Verify both approvers are not the same person (enforced by GitHub)
- Display deployment checklist for approvers to review before approval
- Allow approvers to comment on the pull request with questions or concerns
- Block production deployment until both approvals received

#### Production Gate Conditions

- Runs only after Staging Deploy stage completes successfully
- Requires manual trigger by authorized team member
- Blocks until two distinct approvers approve the deployment
- Blocks if Validation Agent score is below 70 or CRITICAL violations present
- Blocks if security scan shows HIGH or CRITICAL vulnerabilities
- Blocks if any test failure occurred in Test stage
- Blocks if staging environment health checks failed
- Blocks if performance regression exceeds 20% tolerance
- Provides 7-day timeout window for approvals before manual intervention required

#### Production Gate Outputs

- Production deployment approved or rejected status
- Approver identities and approval timestamps recorded
- Deployment checklist reviewed and acknowledged
- Comments from approvers captured for audit trail
- Deployment proceeds to Production Deploy stage if approved
- Deployment blocked and notified to team if rejected

---

### Production Deploy (After Gate Approval)

After successful production gate approval, the production deployment executes with the same steps as Staging Deploy but targeting the production environment.

#### Production Deploy Steps

- Download build artifacts and test artifacts from previous stages
- Pull production-tagged Docker images from container registry
- Download Helm chart tarball from build artifacts
- Configure Terraform backend state file pointing to production AWS account and region
- Initialize Terraform working directory with production configuration
- Run Terraform plan to preview infrastructure changes in production environment
- Require explicit approval for Terraform plan if destructive changes detected
- Apply Terraform infrastructure changes using terraform apply
- Wait for Terraform to complete and verify infrastructure provisioning success
- Deploy Helm chart to production EKS cluster using helm upgrade command
- Apply production-specific Helm values override file (values-production.yaml)
- Verify Helm release status using helm status command
- Run post-deployment health checks against production endpoints
- Verify production backend health at production.domain.com/health endpoint
- Verify production frontend accessibility at production.domain.com
- Verify Keycloak service health in production environment
- Run smoke tests against production API endpoints using production domain
- Verify JWT token validation works with production Keycloak instance
- Verify in-memory state management functions correctly in production
- Generate deployment artifact with Terraform state snapshot, Helm release manifest, and health check results
- Update production deployment status and send success notification

#### Production Deploy Conditions

- Runs only after Production Gate stage receives two approvals
- Fails immediately if Terraform plan shows unexpected changes
- Fails immediately if Terraform apply fails for any reason
- Fails immediately if Helm chart deployment fails
- Fails immediately if health checks fail against production endpoints
- Rolls back failed deployment automatically using Helm rollback and Terraform rollback
- Sends notification to all stakeholders on production deployment success or failure
- Creates production deployment record with all approvers and timestamps

#### Production Deploy Outputs

- Terraform state file updated with production infrastructure
- Helm release deployed to production namespace in EKS cluster
- Production environment fully operational with all services healthy
- Health check results confirming production endpoints responding correctly
- Smoke test results confirming production API functionality
- Deployment artifact with infrastructure state for audit trail

---

## Terraform IaC (AWS)

### Overview

The Terraform infrastructure as code provisions AWS resources for hosting the Modernize Todo App production environment. The Terraform code defines EKS cluster for container orchestration, RDS for persistent data storage, MSK for message streaming, Redis for caching, and supporting networking and security resources. The Terraform layout follows a modular structure with separate modules for each AWS service type, enabling reusable and maintainable infrastructure definitions.

### Terraform Layout

```
terraform/
├── main.tf                 # Root module configuration and provider setup
├── versions.tf             # Terraform and provider version constraints
├── variables.tf            # Input variables for the root module
├── outputs.tf             # Output values from the root module
├── backend.tf             # Backend service configuration (FastAPI)
├── frontend.tf            # Frontend service configuration (nginx + React)
├── keycloak.tf            # Keycloak service configuration
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── eks/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── rds/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── msk/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── redis/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── security_groups/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── iam/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── environments/
│   ├── dev.tfvars         # Development environment variables
│   ├── staging.tfvars      # Staging environment variables
│   └── production.tfvars   # Production environment variables
└── helm/
    └── todo-app/          # Helm chart for todo-app deployment
        ├── Chart.yaml
        ├── values.yaml
        ├── values-dev.yaml
        ├── values-staging.yaml
        ├── values-production.yaml
        └── templates/
            ├── deployment.yaml
            ├── service.yaml
            ├── ingress.yaml
            ├── configmap.yaml
            └── secret.yaml
```

### Terraform Modules

#### VPC Module (modules/vpc/)

The VPC module provisions AWS Virtual Private Cloud resources for network isolation and connectivity. This module creates a custom VPC with public and private subnets, internet gateway for public access, NAT gateways for private subnet outbound internet access, and VPC peering for cross-environment communication.

**Inputs:**
- vpc_cidr_block - CIDR block for the VPC (default 10.0.0.0/16)
- availability_zones - List of availability zones to use (default 2 zones)
- public_subnet_cidrs - CIDR blocks for public subnets
- private_subnet_cidrs - CIDR blocks for private subnets
- enable_vpc_peering - Boolean flag to enable VPC peering (default false)
- peered_vpc_ids - List of VPC IDs to peer with (if peering enabled)

**Outputs:**
- vpc_id - ID of the created VPC
- public_subnet_ids - List of public subnet IDs
- private_subnet_ids - List of private subnet IDs
- internet_gateway_id - ID of the internet gateway
- nat_gateway_ids - List of NAT gateway IDs
- vpc_cidr_block - CIDR block of the VPC
- route_table_ids - List of route table IDs

**Resources:**
- aws_vpc - VPC resource
- aws_subnet - Public and private subnet resources
- aws_internet_gateway - Internet gateway for public internet access
- aws_nat_gateway - NAT gateways for private subnet outbound internet access
- aws_route_table - Route tables for public and private subnets
- aws_route - Route associations for internet gateway and NAT gateways
- aws_vpc_peering_connection - VPC peering connections (if enabled)
- aws_vpc_peering_connection_accepter - VPC peering accepter (if peered)

#### EKS Module (modules/eks/)

The EKS module provisions Amazon Elastic Kubernetes Service for container orchestration. This module creates the EKS cluster, node groups for running containerized workloads, and cluster addons for observability and networking.

**Inputs:**
- cluster_name - Name of the EKS cluster (default todo-app-cluster)
- cluster_version - Kubernetes version for EKS (default 1.28)
- vpc_id - VPC ID from VPC module
- public_subnet_ids - List of public subnet IDs from VPC module
- private_subnet_ids - List of private subnet IDs from VPC module
- node_group_name - Name of the EKS node group (default todo-app-nodes)
- instance_types - List of EC2 instance types for nodes (default t3.medium)
- desired_size - Desired number of worker nodes (default 2)
- min_size - Minimum number of worker nodes (default 2)
- max_size - Maximum number of worker nodes (default 5)
- enable_cluster_autoscaler - Boolean flag to enable cluster autoscaler (default true)
- max_autoscaler_nodes - Maximum nodes when autoscaler enabled (default 10)
- cluster_endpoint_public_access - Boolean for public cluster endpoint (default false)
- cluster_logging_types - List of control plane logging types (default api, audit, authenticator, controllerManager, scheduler)

**Outputs:**
- cluster_id - ID of the EKS cluster
- cluster_endpoint - Endpoint URL for the EKS cluster
- cluster_certificate_authority_data - Cluster CA certificate data for kubeconfig
- cluster_security_group_id - Security group ID for the cluster
- node_group_arn - ARN of the EKS node group
- oidc_provider_arn - ARN of the OIDC provider for IRSA integration
- region - AWS region where cluster is provisioned
- cluster_name - Name of the EKS cluster

**Resources:**
- aws_eks_cluster - EKS cluster resource
- aws_eks_node_group - EKS node group resource
- aws_iam_openid_connect_provider - OIDC provider for IRSA authentication
- aws_eks_addon - Kubernetes addons for VPC CNI, CoreDNS, and kube-proxy
- aws_launch_template - Launch template for worker nodes
- aws_autoscaling_group - Autoscaling group for worker nodes (if autoscaler enabled)

#### RDS Module (modules/rds/)

The RDS module provisions Amazon RDS for persistent data storage. This module creates a PostgreSQL database instance for future data persistence beyond the in-memory storage specified in the constitution. The RDS instance is provisioned as a foundation for future enhancements while maintaining current in-memory storage pattern.

**Inputs:**
- identifier - Identifier for the RDS instance (default todo-app-db)
- engine - Database engine (default postgres)
- engine_version - Database engine version (default 15.4)
- instance_class - RDS instance class for sizing (default db.t3.micro)
- allocated_storage - Allocated storage in GB (default 20)
- storage_type - Storage type (default gp3)
- database_name - Initial database name (default todo_app)
- master_username - Master username for the database
- master_password - Master password for the database (managed via AWS Secrets Manager)
- vpc_id - VPC ID from VPC module
- private_subnet_ids - List of private subnet IDs from VPC module
- multi_az - Boolean flag for Multi-AZ deployment (default true)
- backup_retention_period - Backup retention period in days (default 7)
- backup_window - Daily backup window in UTC (default 03:00-04:00)
- monitoring_interval - Enhanced monitoring interval in seconds (default 60)
- performance_insights_enabled - Boolean flag for Performance Insights (default false)
- deletion_protection - Boolean flag for deletion protection (default true)

**Outputs:**
- db_instance_id - ID of the RDS instance
- db_instance_address - Endpoint address of the RDS instance
- db_instance_port - Port number of the RDS instance (default 5432)
- db_instance_arn - ARN of the RDS instance
- db_name - Name of the initial database
- db_endpoint - Connection endpoint string

**Resources:**
- aws_db_instance - RDS database instance
- aws_db_subnet_group - DB subnet group
- aws_security_group - Security group for RDS access
- aws_db_parameter_group - DB parameter group for configuration
- aws_kms_key - KMS key for encryption at rest
- aws_secretsmanager_secret - Secrets Manager secret for master password

#### MSK Module (modules/msk/)

The MSK module provisions Amazon Managed Streaming for Kafka for event streaming and message queuing. This module creates an MSK cluster with Kafka topics for future event-driven architecture capabilities. The MSK cluster provides a foundation for async processing and event-driven patterns.

**Inputs:**
- cluster_name - Name of the MSK cluster (default todo-app-msk)
- kafka_version - Kafka version (default 3.5.1)
- number_of_broker_nodes - Number of broker nodes (default 2)
- broker_instance_type - EC2 instance type for brokers (default kafka.t3.small)
- broker_volume_size - Volume size per broker in GB (default 100)
- ebs_volume_throughput - EBS volume throughput in MB/s (default 250)
- encryption_at_rest - Boolean flag for encryption at rest (default true)
- client_broker_encryption - Boolean flag for client-broker encryption (default true)
- client_authentication - Client authentication method (default SASL_SCRAM)
- log_retention_hours - CloudWatch Logs retention in hours (default 168)
- vpc_id - VPC ID from VPC module
- private_subnet_ids - List of private subnet IDs from VPC module
- jmx_exporter_enabled - Boolean flag for JMX exporter (default false)
- open_monitoring - Boolean flag for open monitoring (default false)
- zookeeper_enabled - Boolean flag for Zookeeper (default false, using in-broker Zookeeper)

**Outputs:**
- msk_cluster_arn - ARN of the MSK cluster
- msk_cluster_bootstrap_brokers - Comma-separated list of bootstrap brokers
- msk_cluster_zookeeper_connect_string - Zookeeper connection string (if enabled)
- msk_cluster_name - Name of the MSK cluster

**Resources:**
- aws_msk_cluster - MSK cluster resource
- aws_msk_configuration - MSK cluster configuration
- aws_security_group - Security group for MSK access
- aws_cloudwatch_log_group - CloudWatch Logs log group for MSK logs
- aws_kms_key - KMS key for encryption

#### Redis Module (modules/redis/)

The Redis module provisions ElastiCache for Redis for caching and session storage. This module creates a Redis cluster for caching frequently accessed data, storing session tokens, and implementing distributed caching patterns. The Redis cluster supports the existing in-memory storage pattern while providing persistence and scalability.

**Inputs:**
- cluster_id - Identifier for the Redis cluster (default todo-app-redis)
- node_type - Cache node type (default cache.t3.micro)
- num_cache_nodes - Number of cache nodes (default 1)
- engine - Cache engine (default redis)
- engine_version - Cache engine version (default 7.0)
- parameter_group_name - Parameter group name (default default.redis7)
- port - Cache port number (default 6379)
- vpc_id - VPC ID from VPC module
- private_subnet_ids - List of private subnet IDs from VPC module
- automatic_failover_enabled - Boolean flag for automatic failover (default true)
- multi_az_enabled - Boolean flag for Multi-AZ deployment (default true)
- snapshot_retention_limit - Snapshot retention limit (default 5)
- snapshot_window - Daily snapshot window in UTC (default 02:00-03:00)
- at_rest_encryption_enabled - Boolean flag for encryption at rest (default true)
- transit_encryption_enabled - Boolean flag for transit encryption (default true)
- auth_token - Auth token for Redis AUTH (default generated)
- log_delivery_configuration - CloudWatch Logs delivery configuration

**Outputs:**
- cluster_id - ID of the Redis cluster
- cluster_address - Connection endpoint for the Redis cluster
- cluster_port - Port number of the Redis cluster
- cluster_engine - Cache engine version
- cluster_arn - ARN of the Redis cluster
- cache_nodes - List of cache node information

**Resources:**
- aws_elasticache_replication_group - ElastiCache replication group
- aws_elasticache_subnet_group - Subnet group for Redis
- aws_security_group - Security group for Redis access
- aws_elasticache_parameter_group - Parameter group for configuration
- aws_cloudwatch_log_group - CloudWatch Logs log group

#### Security Groups Module (modules/security_groups/)

The Security Groups module provisions AWS security groups and rules for network access control. This module creates security groups for the EKS cluster, RDS, MSK, Redis, and services, with rules allowing necessary traffic while restricting unauthorized access.

**Inputs:**
- vpc_id - VPC ID from VPC module
- vpc_cidr_block - CIDR block of the VPC
- eks_cluster_sg_id - Security group ID for EKS cluster (from EKS module)
- rds_sg_id - Security group ID for RDS (from RDS module)
- msk_sg_id - Security group ID for MSK (from MSK module)
- redis_sg_id - Security group ID for Redis (from Redis module)
- allowed_cidr_blocks - List of CIDR blocks allowed to access services (default VPC CIDR)
- ingress_ports - List of ingress port configurations
- egress_enabled - Boolean flag to allow egress traffic (default true)

**Outputs:**
- security_group_ids - List of all security group IDs
- security_group_arns - List of all security group ARNs

**Resources:**
- aws_security_group - Security groups for each service
- aws_vpc_security_group_ingress - Ingress rules
- aws_vpc_security_group_egress - Egress rules (if enabled)

#### IAM Module (modules/iam/)

The IAM module provisions AWS IAM roles and policies for EKS worker nodes and service accounts. This module creates IAM roles with least privilege access to EKS nodes, RDS, MSK, Redis, and other AWS resources required by the application.

**Inputs:**
- cluster_name - Name of the EKS cluster (from EKS module)
- oidc_provider_arn - ARN of the OIDC provider (from EKS module)
- region - AWS region
- eks_cluster_role_name - Name of the EKS cluster IAM role (default todo-app-eks-cluster-role)
- node_role_name - Name of the node IAM role (default todo-app-eks-node-role)
- rds_arn - ARN of RDS instance for policy attachment
- msk_arn - ARN of MSK cluster for policy attachment
- redis_arn - ARN of Redis cluster for policy attachment

**Outputs:**
- eks_cluster_role_arn - ARN of the EKS cluster IAM role
- eks_cluster_role_name - Name of the EKS cluster IAM role
- node_role_arn - ARN of the node IAM role
- node_role_name - Name of the node IAM role

**Resources:**
- aws_iam_role - IAM roles for EKS cluster and nodes
- aws_iam_role_policy_attachment - Policy attachments
- aws_iam_policy - Inline IAM policies for least privilege access

### Terraform Root Module

#### Main Configuration (main.tf)

The main.tf file configures Terraform providers, calls submodules, and provisions backend, frontend, and Keycloak Kubernetes resources.

**Providers:**
- AWS provider configured with region and backend state
- Helm provider configured for Kubernetes cluster access
- Kubernetes provider configured for direct cluster resource management
- Random provider for generating unique resource names

**Module Calls:**
- VPC module called with variables from environment-specific tfvars file
- EKS module called with outputs from VPC module
- RDS module called with outputs from VPC module
- MSK module called with outputs from VPC module
- Redis module called with outputs from VPC module
- Security Groups module called with outputs from VPC module and other modules
- IAM module called with outputs from EKS module and other resources
- Helm release for backend service using todo-app Helm chart
- Helm release for frontend service using todo-app Helm chart
- Helm release for Keycloak service using Keycloak Helm chart

**Backend Service (backend.tf):**
- Kubernetes Deployment resource for FastAPI backend
- Kubernetes Service resource for backend (ClusterIP type for internal access, LoadBalancer type for external access if needed)
- Kubernetes ConfigMap resource for backend environment variables (PORT, JSON_PLACEHOLDER_URL, Keycloak settings)
- Kubernetes Secret resource for sensitive backend configuration (Keycloak client secret)
- Kubernetes HorizontalPodAutoscaler for backend pods (target CPU utilization 70%, min 2 replicas, max 10 replicas)
- Kubernetes PodDisruptionBudget for backend (minimum available pods 2, maximum unavailable 1)

**Frontend Service (frontend.tf):**
- Kubernetes Deployment resource for nginx frontend
- Kubernetes Service resource for frontend (LoadBalancer type for external access)
- Kubernetes Ingress resource for frontend (domain routing, SSL termination via AWS ALB)
- Kubernetes ConfigMap resource for nginx configuration
- Kubernetes HorizontalPodAutoscaler for frontend pods (target CPU utilization 70%, min 2 replicas, max 10 replicas)
- Kubernetes PodDisruptionBudget for frontend (minimum available pods 2, maximum unavailable 1)

**Keycloak Service (keycloak.tf):**
- Kubernetes Deployment resource for Keycloak
- Kubernetes Service resource for Keycloak (LoadBalancer type for external access)
- Kubernetes Ingress resource for Keycloak (domain routing, SSL termination via AWS ALB)
- Kubernetes ConfigMap resource for Keycloak environment variables (admin credentials, realm settings)
- Kubernetes Secret resource for Keycloak admin password and client secret
- Kubernetes PersistentVolumeClaim for Keycloak database (if using PostgreSQL for Keycloak)
- StatefulSet for Keycloak with persistent storage (recommended for production)

#### Variables (variables.tf)

The variables.tf file defines input variables for the root module with default values and descriptions.

**Variables:**
- aws_region - AWS region for infrastructure (default us-east-1)
- environment_name - Environment name for resource naming (dev, staging, production)
- project_name - Project name prefix for resources (default todo-app)
- vpc_cidr_block - CIDR block for VPC (default 10.0.0.0/16)
- availability_zones - Availability zones to use (default us-east-1a, us-east-1b)
- enable_vpc_peering - Enable VPC peering (default false)
- cluster_version - Kubernetes version for EKS (default 1.28)
- node_instance_type - EC2 instance type for worker nodes (default t3.medium)
- node_desired_size - Desired number of worker nodes (default 2)
- enable_cluster_autoscaler - Enable cluster autoscaler (default true)
- rds_instance_class - RDS instance class (default db.t3.micro)
- rds_allocated_storage - Allocated storage in GB (default 20)
- enable_rds - Boolean flag to provision RDS (default true)
- msk_broker_instance_type - MSK broker instance type (default kafka.t3.small)
- msk_broker_volume_size - MSK broker volume size in GB (default 100)
- enable_msk - Boolean flag to provision MSK (default true)
- redis_node_type - Redis cache node type (default cache.t3.micro)
- redis_num_cache_nodes - Number of cache nodes (default 1)
- enable_redis - Boolean flag to provision Redis (default true)
- backend_replica_count - Number of backend pod replicas (default 2)
- frontend_replica_count - Number of frontend pod replicas (default 2)
- keycloak_replica_count - Number of Keycloak pod replicas (default 1, use StatefulSet for production)
- domain_name - Domain name for Ingress resources (default todo-app.example.com)
- backend_port - Backend service port (default 3001)
- frontend_port - Frontend service port (default 80)
- keycloak_http_port - Keycloak HTTP port (default 8080)
- keycloak_https_port - Keycloak HTTPS port (default 8443)
- enable_istio - Boolean flag to enable Istio service mesh (default true)
- monitoring_enabled - Boolean flag to enable CloudWatch monitoring (default true)

#### Outputs (outputs.tf)

The outputs.tf file defines output values from the root module for consumption by other tools or modules.

**Outputs:**
- vpc_id - VPC ID from VPC module
- cluster_id - EKS cluster ID from EKS module
- cluster_endpoint - EKS cluster endpoint from EKS module
- cluster_certificate_authority_data - Cluster CA certificate data for kubeconfig generation
- node_group_arn - EKS node group ARN from EKS module
- rds_endpoint - RDS connection endpoint from RDS module
- rds_instance_id - RDS instance ID from RDS module
- msk_bootstrap_brokers - MSK bootstrap brokers from MSK module
- msk_cluster_arn - MSK cluster ARN from MSK module
- redis_endpoint - Redis connection endpoint from Redis module
- redis_cluster_id - Redis cluster ID from Redis module
- backend_service_url - Backend service URL (constructed from Ingress and domain)
- frontend_service_url - Frontend service URL (constructed from Ingress and domain)
- keycloak_service_url - Keycloak service URL (constructed from Ingress and domain)
- kubeconfig_command - Command to generate kubeconfig file for cluster access
- backend_load_balancer_dns - Backend LoadBalancer DNS name (if LoadBalancer type service)
- frontend_load_balancer_dns - Frontend LoadBalancer DNS name
- keycloak_load_balancer_dns - Keycloak LoadBalancer DNS name

#### Backend State Configuration

The Terraform backend state is managed separately for each environment to prevent cross-environment state conflicts.

**State Files:**
- terraform.tfstate.dev - State file for development environment
- terraform.tfstate.staging - State file for staging environment
- terraform.tfstate.production - State file for production environment
- terraform.tfstate.backup.dev - State backup for development environment
- terraform.tfstate.backup.staging - State backup for staging environment
- terraform.tfstate.backup.production - State backup for production environment

**Backend Configuration:**
- terraform/dev.backend.tfvars - Backend configuration for development (local state file, S3 backend, or Terraform Cloud)
- terraform/staging.backend.tfvars - Backend configuration for staging (S3 backend recommended)
- terraform/production.backend.tfvars - Backend configuration for production (S3 backend with state locking required)

### Environment-Specific Variables

#### Development Variables (environments/dev.tfvars)

Development environment variables prioritize cost optimization and fast iteration over high availability and performance.

**Variables:**
- aws_region = us-east-1
- environment_name = dev
- vpc_cidr_block = 10.0.0.0/16
- availability_zones = [us-east-1a]
- enable_vpc_peering = false
- cluster_version = 1.28
- node_instance_type = t3.small
- node_desired_size = 1
- enable_cluster_autoscaler = false
- rds_instance_class = db.t3.micro
- rds_allocated_storage = 20
- enable_rds = false (in-memory storage pattern maintained)
- msk_broker_instance_type = kafka.t3.small
- msk_broker_volume_size = 50
- enable_msk = false
- redis_node_type = cache.t3.micro
- redis_num_cache_nodes = 1
- enable_redis = false
- backend_replica_count = 1
- frontend_replica_count = 1
- keycloak_replica_count = 1
- domain_name = dev.todo-app.example.com
- enable_istio = false
- monitoring_enabled = false

#### Staging Variables (environments/staging.tfvars)

Staging environment variables balance cost with high availability and performance for pre-production validation.

**Variables:**
- aws_region = us-east-1
- environment_name = staging
- vpc_cidr_block = 10.1.0.0/16
- availability_zones = [us-east-1a, us-east-1b]
- enable_vpc_peering = false
- cluster_version = 1.28
- node_instance_type = t3.medium
- node_desired_size = 2
- enable_cluster_autoscaler = true
- rds_instance_class = db.t3.micro
- rds_allocated_storage = 20
- enable_rds = true (foundation for future data persistence)
- msk_broker_instance_type = kafka.t3.small
- msk_broker_volume_size = 100
- enable_msk = true (foundation for future event-driven architecture)
- redis_node_type = cache.t3.micro
- redis_num_cache_nodes = 1
- enable_redis = true (caching layer for in-memory storage)
- backend_replica_count = 2
- frontend_replica_count = 2
- keycloak_replica_count = 1
- domain_name = staging.todo-app.example.com
- enable_istio = true
- monitoring_enabled = true

#### Production Variables (environments/production.tfvars)

Production environment variables prioritize high availability, performance, security, and scalability over cost optimization.

**Variables:**
- aws_region = us-east-1
- environment_name = production
- vpc_cidr_block = 10.2.0.0/16
- availability_zones = [us-east-1a, us-east-1b, us-east-1c]
- enable_vpc_peering = true (for cross-environment VPC peering)
- cluster_version = 1.28
- node_instance_type = t3.medium
- node_desired_size = 3
- enable_cluster_autoscaler = true
- rds_instance_class = db.t3.medium
- rds_allocated_storage = 100
- enable_rds = true (foundation for data persistence beyond in-memory)
- msk_broker_instance_type = kafka.t3.medium
- msk_broker_volume_size = 200
- enable_msk = true (foundation for event-driven architecture)
- redis_node_type = cache.t3.medium
- redis_num_cache_nodes = 2 (Redis cluster for high availability)
- enable_redis = true (distributed caching layer)
- backend_replica_count = 3
- frontend_replica_count = 3
- keycloak_replica_count = 1 (use StatefulSet with persistent storage)
- domain_name = todo-app.example.com
- enable_istio = true (with mTLS enabled)
- monitoring_enabled = true

---

## Helm Charts

### Overview

The Helm chart defines the Kubernetes application deployment specifications for the Todo App. The Helm chart encapsulates Kubernetes resources for the FastAPI backend, React frontend with nginx, and Keycloak identity provider, enabling environment-specific configuration through values files.

### Helm Chart Structure

```
helm/todo-app/
├── Chart.yaml             # Chart metadata (name, version, description)
├── values.yaml             # Default values for all environments
├── values-dev.yaml         # Dev environment overrides
├── values-staging.yaml    # Staging environment overrides
├── values-production.yaml  # Production environment overrides
└── templates/
    ├── NOTES.txt            # Helm release notes displayed after install
    ├── _helpers.tpl          # Template helpers for common patterns
    ├── backend/
    │   ├── deployment.yaml   # Backend deployment manifest
    │   ├── service.yaml      # Backend service manifest
    │   ├── hpa.yaml         # Backend horizontal pod autoscaler
    │   ├── pdb.yaml         # Backend pod disruption budget
    │   ├── configmap.yaml   # Backend environment configuration
    │   └── secret.yaml      # Backend sensitive configuration
    ├── frontend/
    │   ├── deployment.yaml   # Frontend deployment manifest
    │   ├── service.yaml      # Frontend service manifest
    │   ├── ingress.yaml     # Frontend ingress for domain routing
    │   ├── hpa.yaml         # Frontend horizontal pod autoscaler
    │   ├── pdb.yaml         # Frontend pod disruption budget
    │   └── configmap.yaml   # Nginx configuration
    └── keycloak/
        ├── deployment.yaml   # Keycloak deployment manifest
        ├── service.yaml      # Keycloak service manifest
        ├── ingress.yaml     # Keycloak ingress for domain routing
        ├── statefulset.yaml # Keycloak statefulset (production only)
        ├── configmap.yaml   # Keycloak environment variables
        ├── secret.yaml      # Keycloak sensitive configuration
        ├── pvc.yaml         # Persistent volume claim for Keycloak database
        └── svc.yaml         # Service for Keycloak database
```

### Chart Metadata (Chart.yaml)

The Chart.yaml file defines Helm chart metadata.

**Metadata:**
- apiVersion: v2
- name: todo-app
- description: Helm chart for Modernize Todo App (FastAPI backend, React frontend, Keycloak identity provider)
- type: application
- version: 1.0.0
- appVersion: 1.0.0
- keywords: todo, fastapi, react, keycloak, kubernetes
- maintainers: InfiniAI team
- icon: URL to chart icon (optional)

### Default Values (values.yaml)

The values.yaml file defines default configuration values applicable to all environments.

**Global Values:**
- replicaCount: 2 (default number of replicas for all services)
- imagePullPolicy: IfNotPresent (image pull policy)
- imagePullSecrets: Empty list (secret names for private registries)
- nameOverride: Empty string (override release name)
- fullnameOverride: Empty string (override full qualified release name)
- revisionHistoryLimit: 10 (number of old replica sets to retain)
- serviceAccount.create: true (create service account)
- serviceAccount.name: Empty string (use default service account name)
- serviceAccount.annotations: Empty map (service account annotations)
- podSecurityContext.enabled: false (enable pod security context)
- podSecurityContext.fsGroup: Empty string (file system group ID)
- podSecurityContext.supplementalGroups: Empty list (supplemental groups)

**Backend Values:**
- backend.enabled: true (enable backend deployment)
- backend.replicaCount: 2 (backend pod replicas)
- backend.image.repository: your-registry.com/todo-app-backend (backend image repository)
- backend.image.tag: latest (backend image tag)
- backend.image.pullPolicy: IfNotPresent (backend image pull policy)
- backend.service.type: ClusterIP (backend service type)
- backend.service.port: 3001 (backend service port)
- backend.service.annotations: Empty map (backend service annotations)
- backend.env.PORT: 3001 (backend environment port)
- backend.env.JSON_PLACEHOLDER_URL: https://jsonplaceholder.typicode.com/todos (JSONPlaceholder URL)
- backend.env.KEYCLOAK_URL: Empty string (Keycloak URL, set per environment)
- backend.env.KEYCLOAK_REALM: todo-app (Keycloak realm name)
- backend.env.KEYCLOAK_CLIENT_ID: todo-backend (Keycloak client ID)
- backend.env.REDIS_ENABLED: false (Redis caching enabled flag)
- backend.env.REDIS_HOST: Empty string (Redis host)
- backend.env.REDIS_PORT: 6379 (Redis port)
- backend.resources.requests.cpu: 100m (backend CPU request)
- backend.resources.requests.memory: 128Mi (backend memory request)
- backend.resources.limits.cpu: 500m (backend CPU limit)
- backend.resources.limits.memory: 512Mi (backend memory limit)
- backend.livenessProbe.enabled: true (enable liveness probe)
- backend.livenessProbe.httpGet.path: /health (liveness probe path)
- backend.livenessProbe.httpGet.port: 3001 (liveness probe port)
- backend.livenessProbe.initialDelaySeconds: 10 (liveness probe initial delay)
- backend.livenessProbe.periodSeconds: 30 (liveness probe period)
- backend.livenessProbe.successThreshold: 1 (liveness probe success threshold)
- backend.livenessProbe.failureThreshold: 3 (liveness probe failure threshold)
- backend.readinessProbe.enabled: true (enable readiness probe)
- backend.readinessProbe.httpGet.path: /health (readiness probe path)
- backend.readinessProbe.httpGet.port: 3001 (readiness probe port)
- backend.readinessProbe.initialDelaySeconds: 5 (readiness probe initial delay)
- backend.readinessProbe.periodSeconds: 10 (readiness probe period)
- backend.readinessProbe.successThreshold: 1 (readiness probe success threshold)
- backend.readinessProbe.failureThreshold: 3 (readiness probe failure threshold)
- backend.nodeSelector: Empty map (backend node selector)
- backend.tolerations: Empty list (backend tolerations)
- backend.affinity: Empty map (backend affinity)
- backend.topologySpreadConstraints: Empty list (backend topology spread constraints)
- backend.podAnnotations: Empty map (backend pod annotations)
- backend.podLabels: Empty map (backend pod labels)
- backend.hpa.enabled: true (enable horizontal pod autoscaler)
- backend.hpa.minReplicas: 2 (minimum backend replicas)
- backend.hpa.maxReplicas: 10 (maximum backend replicas)
- backend.hpa.targetCPUUtilizationPercentage: 70 (target CPU utilization)
- backend.pdb.enabled: true (enable pod disruption budget)
- backend.pdb.minAvailable: 2 (minimum available pods)
- backend.pdb.maxUnavailable: 1 (maximum unavailable pods)

**Frontend Values:**
- frontend.enabled: true (enable frontend deployment)
- frontend.replicaCount: 2 (frontend pod replicas)
- frontend.image.repository: your-registry.com/todo-app-frontend (frontend image repository)
- frontend.image.tag: latest (frontend image tag)
- frontend.image.pullPolicy: IfNotPresent (frontend image pull policy)
- frontend.service.type: LoadBalancer (frontend service type)
- frontend.service.port: 80 (frontend service port)
- frontend.service.annotations: Empty map (frontend service annotations)
- frontend.ingress.enabled: true (enable ingress)
- frontend.ingress.className: nginx (ingress class)
- frontend.ingress.annotations: Empty map (ingress annotations)
- frontend.ingress.hosts: Empty list (hosts set per environment)
- frontend.ingress.paths: [ / ] (ingress paths)
- frontend.ingress.tls: Empty map (TLS configuration set per environment)
- frontend.resources.requests.cpu: 100m (frontend CPU request)
- frontend.resources.requests.memory: 64Mi (frontend memory request)
- frontend.resources.limits.cpu: 300m (frontend CPU limit)
- frontend.resources.limits.memory: 128Mi (frontend memory limit)
- frontend.livenessProbe.enabled: true (enable liveness probe)
- frontend.livenessProbe.httpGet.path: / (liveness probe path)
- frontend.livenessProbe.httpGet.port: 80 (liveness probe port)
- frontend.livenessProbe.initialDelaySeconds: 10 (liveness probe initial delay)
- frontend.livenessProbe.periodSeconds: 30 (liveness probe period)
- frontend.livenessProbe.successThreshold: 1 (liveness probe success threshold)
- frontend.livenessProbe.failureThreshold: 3 (liveness probe failure threshold)
- frontend.readinessProbe.enabled: true (enable readiness probe)
- frontend.readinessProbe.httpGet.path: / (readiness probe path)
- frontend.readinessProbe.httpGet.port: 80 (readiness probe port)
- frontend.readinessProbe.initialDelaySeconds: 5 (readiness probe initial delay)
- frontend.readinessProbe.periodSeconds: 10 (readiness probe period)
- frontend.readinessProbe.successThreshold: 1 (readiness probe success threshold)
- frontend.readinessProbe.failureThreshold: 3 (readiness probe failure threshold)
- frontend.nodeSelector: Empty map (frontend node selector)
- frontend.tolerations: Empty list (frontend tolerations)
- frontend.affinity: Empty map (frontend affinity)
- frontend.topologySpreadConstraints: Empty list (frontend topology spread constraints)
- frontend.podAnnotations: Empty map (frontend pod annotations)
- frontend.podLabels: Empty map (frontend pod labels)
- frontend.hpa.enabled: true (enable horizontal pod autoscaler)
- frontend.hpa.minReplicas: 2 (minimum frontend replicas)
- frontend.hpa.maxReplicas: 10 (maximum frontend replicas)
- frontend.hpa.targetCPUUtilizationPercentage: 70 (target CPU utilization)
- frontend.pdb.enabled: true (enable pod disruption budget)
- frontend.pdb.minAvailable: 2 (minimum available pods)
- frontend.pdb.maxUnavailable: 1 (maximum unavailable pods)

**Keycloak Values:**
- keycloak.enabled: true (enable Keycloak deployment)
- keycloak.replicaCount: 1 (Keycloak pod replicas)
- keycloak.image.repository: quay.io/keycloak/keycloak (Keycloak image repository)
- keycloak.image.tag: 25.0 (Keycloak image version)
- keycloak.image.pullPolicy: IfNotPresent (Keycloak image pull policy)
- keycloak.service.type: LoadBalancer (Keycloak service type)
- keycloak.service.httpPort: 8080 (Keycloak HTTP port)
- keycloak.service.httpsPort: 8443 (Keycloak HTTPS port)
- keycloak.service.annotations: Empty map (Keycloak service annotations)
- keycloak.ingress.enabled: true (enable ingress)
- keycloak.ingress.className: nginx (ingress class)
- keycloak.ingress.annotations: Empty map (ingress annotations)
- keycloak.ingress.hosts: Empty list (hosts set per environment)
- keycloak.ingress.paths: [ / ] (ingress paths)
- keycloak.ingress.tls: Empty map (TLS configuration set per environment)
- keycloak.env.KEYCLOAK_ADMIN: admin (Keycloak admin username)
- keycloak.env.DB_VENDOR: postgres (Keycloak database vendor)
- keycloak.env.DB_ADDR: Empty string (Keycloak database address)
- keycloak.env.DB_USER: keycloak (Keycloak database user)
- keycloak.env.DB_PASSWORD: Empty string (set via secret)
- keycloak.env.JAVA_OPTS_APPEND: -Xmx512m -XX:MaxMetaspaceSize=256m (JVM options)
- keycloak.pvc.enabled: false (enable persistent volume claim)
- keycloak.pvc.storageClass: gp2 (storage class)
- keycloak.pvc.size: 1Gi (volume size)
- keycloak.statefulset.enabled: false (enable StatefulSet for production)
- keycloak.statefulset.serviceName: postgresql (StatefulSet service name)
- keycloak.resources.requests.cpu: 500m (Keycloak CPU request)
- keycloak.resources.requests.memory: 512Mi (Keycloak memory request)
- keycloak.resources.limits.cpu: 2000m (Keycloak CPU limit)
- keycloak.resources.limits.memory: 2Gi (Keycloak memory limit)
- keycloak.livenessProbe.enabled: true (enable liveness probe)
- keycloak.livenessProbe.httpGet.path: /health/ready (liveness probe path)
- keycloak.livenessProbe.httpGet.port: 8080 (liveness probe port)
- keycloak.livenessProbe.initialDelaySeconds: 60 (liveness probe initial delay)
- keycloak.livenessProbe.periodSeconds: 30 (liveness probe period)
- keycloak.livenessProbe.successThreshold: 1 (liveness probe success threshold)
- keycloak.livenessProbe.failureThreshold: 3 (liveness probe failure threshold)
- keycloak.readinessProbe.enabled: true (enable readiness probe)
- keycloak.readinessProbe.httpGet.path: /health/ready (readiness probe path)
- keycloak.readinessProbe.httpGet.port: 8080 (readiness probe port)
- keycloak.readinessProbe.initialDelaySeconds: 30 (readiness probe initial delay)
- keycloak.readinessProbe.periodSeconds: 10 (readiness probe period)
- keycloak.readinessProbe.successThreshold: 1 (readiness probe success threshold)
- keycloak.readinessProbe.failureThreshold: 3 (readiness probe failure threshold)

### Dev Environment Overrides (values-dev.yaml)

The values-dev.yaml file overrides default values for development environment configuration.

**Global Overrides:**
- replicaCount: 1 (single replica for development)
- imagePullPolicy: Always (always pull images for development)

**Backend Overrides:**
- backend.replicaCount: 1 (single backend replica)
- backend.service.type: NodePort (NodePort service for local access)
- backend.image.tag: dev (use dev image tag)
- backend.hpa.enabled: false (disable autoscaling in development)
- backend.pdb.enabled: false (disable pod disruption budget in development)
- backend.env.REDIS_ENABLED: false (Redis disabled in development)
- backend.env.KEYCLOAK_URL: http://keycloak:8080 (use service name for Keycloak URL in development)

**Frontend Overrides:**
- frontend.replicaCount: 1 (single frontend replica)
- frontend.service.type: NodePort (NodePort service for local access)
- frontend.image.tag: dev (use dev image tag)
- frontend.ingress.enabled: false (disable ingress in development)
- frontend.hpa.enabled: false (disable autoscaling in development)
- frontend.pdb.enabled: false (disable pod disruption budget in development)

**Keycloak Overrides:**
- keycloak.replicaCount: 1 (single Keycloak replica)
- keycloak.service.type: NodePort (NodePort service for local access)
- keycloak.ingress.enabled: false (disable ingress in development)
- keycloak.pvc.enabled: true (enable PVC for Keycloak database in development)
- keycloak.pvc.size: 1Gi (1Gi volume for development)
- keycloak.statefulset.enabled: false (use Deployment in development)
- keycloak.env.DB_ADDR: Empty string (use local PostgreSQL)
- keycloak.resources.requests.cpu: 250m (reduced CPU request)
- keycloak.resources.requests.memory: 256Mi (reduced memory request)

### Staging Environment Overrides (values-staging.yaml)

The values-staging.yaml file overrides default values for staging environment configuration.

**Global Overrides:**
- replicaCount: 2 (two replicas for staging)

**Backend Overrides:**
- backend.replicaCount: 2 (two backend replicas)
- backend.service.type: ClusterIP (ClusterIP service for internal access)
- backend.image.tag: staging (use staging image tag)
- backend.env.REDIS_ENABLED: true (Redis enabled in staging)
- backend.env.REDIS_HOST: redis-service (Redis service name in Kubernetes)
- backend.env.KEYCLOAK_URL: https://staging-keycloak.todo-app.example.com (staging Keycloak URL)
- backend.hpa.minReplicas: 2 (minimum 2 replicas)
- backend.hpa.maxReplicas: 5 (maximum 5 replicas in staging)

**Frontend Overrides:**
- frontend.replicaCount: 2 (two frontend replicas)
- frontend.image.tag: staging (use staging image tag)
- frontend.ingress.hosts: [staging.todo-app.example.com] (staging domain)
- frontend.ingress.tls.enabled: true (enable TLS)
- frontend.ingress.tls.secretName: todo-app-tls-cert (TLS secret name)
- frontend.hpa.minReplicas: 2 (minimum 2 replicas)
- frontend.hpa.maxReplicas: 5 (maximum 5 replicas in staging)

**Keycloak Overrides:**
- keycloak.image.tag: 25.0-stable (use stable Keycloak version)
- keycloak.ingress.hosts: [staging-keycloak.todo-app.example.com] (staging Keycloak domain)
- keycloak.ingress.tls.enabled: true (enable TLS)
- keycloak.ingress.tls.secretName: todo-app-keycloak-tls-cert (TLS secret name)
- keycloak.pvc.enabled: true (enable PVC for Keycloak database)
- keycloak.pvc.size: 5Gi (5Gi volume for staging)
- keycloak.statefulset.enabled: false (use Deployment in staging)
- keycloak.env.DB_ADDR: Empty string (use local PostgreSQL with PVC)

### Production Environment Overrides (values-production.yaml)

The values-production.yaml file overrides default values for production environment configuration with high availability and security.

**Global Overrides:**
- replicaCount: 3 (three replicas for production)
- podSecurityContext.enabled: true (enable pod security context)
- podSecurityContext.fsGroup: 2000 (file system group ID)

**Backend Overrides:**
- backend.replicaCount: 3 (three backend replicas for production)
- backend.service.type: ClusterIP (ClusterIP service for internal access)
- backend.image.tag: production (use production image tag)
- backend.env.REDIS_ENABLED: true (Redis enabled in production)
- backend.env.REDIS_HOST: redis-cluster.production.svc.cluster.local (Redis cluster service in production)
- backend.env.KEYCLOAK_URL: https://keycloak.todo-app.example.com (production Keycloak URL)
- backend.resources.requests.cpu: 200m (increased CPU request)
- backend.resources.requests.memory: 256Mi (increased memory request)
- backend.resources.limits.cpu: 1000m (increased CPU limit)
- backend.resources.limits.memory: 1Gi (increased memory limit)
- backend.hpa.minReplicas: 3 (minimum 3 replicas)
- backend.hpa.maxReplicas: 10 (maximum 10 replicas in production)
- backend.hpa.targetCPUUtilizationPercentage: 75 (target CPU utilization)

**Frontend Overrides:**
- frontend.replicaCount: 3 (three frontend replicas for production)
- frontend.image.tag: production (use production image tag)
- frontend.ingress.hosts: [todo-app.example.com, www.todo-app.example.com] (production domains)
- frontend.ingress.tls.enabled: true (enable TLS)
- frontend.ingress.tls.secretName: todo-app-tls-cert (TLS secret name)
- frontend.ingress.annotations.cert-manager.io/cluster-issuer: letsencrypt-prod (use Let's Encrypt production)
- frontend.resources.requests.cpu: 200m (increased CPU request)
- frontend.resources.requests.memory: 128Mi (increased memory request)
- frontend.resources.limits.cpu: 500m (increased CPU limit)
- frontend.resources.limits.memory: 256Mi (increased memory limit)
- frontend.hpa.minReplicas: 3 (minimum 3 replicas)
- frontend.hpa.maxReplicas: 10 (maximum 10 replicas in production)
- frontend.hpa.targetCPUUtilizationPercentage: 75 (target CPU utilization)

**Keycloak Overrides:**
- keycloak.image.tag: 25.0.0 (use specific Keycloak version)
- keycloak.replicaCount: 1 (StatefulSet manages replicas)
- keycloak.ingress.hosts: [keycloak.todo-app.example.com, auth.todo-app.example.com] (production Keycloak domains)
- keycloak.ingress.tls.enabled: true (enable TLS)
- keycloak.ingress.tls.secretName: todo-app-keycloak-tls-cert (TLS secret name)
- keycloak.ingress.annotations.cert-manager.io/cluster-issuer: letsencrypt-prod (use Let's Encrypt production)
- keycloak.pvc.enabled: false (StatefulSet manages persistent storage)
- keycloak.statefulset.enabled: true (use StatefulSet for production with persistent storage)
- keycloak.statefulset.volumeClaimTemplates.size: 20Gi (20Gi persistent volume for production)
- keycloak.statefulset.volumeClaimTemplates.storageClass: gp2 (gp2 storage class)
- keycloak.resources.requests.cpu: 1000m (increased CPU request)
- keycloak.resources.requests.memory: 1Gi (increased memory request)
- keycloak.resources.limits.cpu: 4000m (increased CPU limit)
- keycloak.resources.limits.memory: 4Gi (increased memory limit)

---

## Istio mTLS

### Overview

Istio mutual TLS (mTLS) provides service-to-service encryption and authentication within the Kubernetes cluster. The mTLS configuration ensures all traffic between services is encrypted and authenticated using certificates managed by Istio. This enhances security by preventing unauthorized service access and protecting data in transit.

### PeerAuthentication Configuration

The PeerAuthentication resource defines mTLS enforcement at the workload level using STRICT or PERMISSIVE modes.

#### Global PeerAuthentication (STRICT Mode)

The global PeerAuthentication configuration enforces mTLS for all workloads in the cluster using STRICT mode. This configuration applies to development, staging, and production environments to ensure consistent mTLS enforcement.

**Configuration:**
- mode: STRICT (enforce mTLS for all peer authentication)
- mtls: Enabled (mutual TLS enabled)

**Selectors:**
- mode: STRICT applies to all workloads without explicit selector (global default)

**Port-Level mTLS:**
- All ports on all workloads require mTLS authentication

**Namespace Scope:**
- Applies to the default namespace where all application services are deployed

**PeerAuthentication Resource Definition:**
- apiVersion: security.istio.io/v1beta1
- kind: PeerAuthentication
- metadata.name: default-strict-mtls
- metadata.namespace: default (or application-specific namespace)
- spec.mtls.mode: STRICT (require mTLS for all connections)

#### Permissive Mode (Migration Path)

The PERMISSIVE mode allows services to accept both mTLS and plain text traffic during migration to mTLS. This configuration is useful for gradual rollout or when integrating with services outside the mesh.

**Configuration:**
- mode: PERMISSIVE (allow both mTLS and plain text)
- mtls: Enabled (mutual TLS enabled but not required)

**Selectors:**
- mode: PERMISSIVE applies to specific workloads or namespaces

**Use Cases:**
- Legacy services not yet adapted for mTLS
- External services connecting to mesh services
- Gradual migration path from plain text to mTLS

**PeerAuthentication Resource Definition:**
- apiVersion: security.istio.io/v1beta1
- kind: PeerAuthentication
- metadata.name: permissive-mtls
- metadata.namespace: external (or namespace with legacy services)
- spec.mtls.mode: PERMISSIVE (allow both mTLS and plain text)

### PeerAuthentication Scope

#### Namespace-Level Scope

Namespace-level PeerAuthentication applies to all workloads within a specific namespace. This provides a way to enforce mTLS policies per namespace rather than globally.

**Configuration:**
- mode: STRICT (namespace-wide mTLS enforcement)
- mtls: Enabled

**Namespace Applications:**
- default namespace: Enforces STRICT mTLS for all application services
- kube-system namespace: May use PERMISSIVE for system services
- external namespace: Uses PERMISSIVE for external integrations

**Namespace-Level Resource Definition:**
- apiVersion: security.istio.io/v1beta1
- kind: PeerAuthentication
- metadata.name: namespace-strict-mtls
- metadata.namespace: application-specific (e.g., todo-app)
- spec.mtls.mode: STRICT

#### Workload-Level Scope

Workload-level PeerAuthentication allows fine-grained control over mTLS enforcement for specific services or pods. This scope is useful when individual services require different mTLS policies.

**Selectors:**
- mode: STRICT or PERMISSIVE
- selector.matchLabels: Map of labels to select specific workloads
- selector.matchExpressions: List of label selectors for complex selection

**Workload-Level Examples:**
- Backend service: STRICT mTLS (FastAPI backend requires secure service-to-service communication)
- Frontend service: STRICT mTLS (nginx frontend requires secure service-to-service communication)
- Keycloak service: STRICT mTLS (Keycloak requires secure service-to-service communication)
- External API integration: PERMISSIVE mTLS (allows plain text with external JSONPlaceholder)

**Workload-Level Resource Definition:**
- apiVersion: security.istio.io/v1beta1
- kind: PeerAuthentication
- metadata.name: backend-strict-mtls
- metadata.namespace: default
- spec.selector.matchLabels.app: backend (select backend workload)
- spec.mtls.mode: STRICT

### Istio Gateway Configuration

The Istio Gateway configuration defines the ingress gateway for external access to mesh services. The gateway is configured with TLS settings to terminate SSL at the gateway while maintaining mTLS within the mesh.

**Gateway Configuration:**
- apiVersion: networking.istio.io/v1beta1
- kind: Gateway
- metadata.name: todo-app-gateway
- metadata.namespace: istio-system (or application namespace)
- spec.servers.port.number: 80 (HTTP port)
- spec.servers.port.name: http
- spec.servers.port.protocol: HTTP
- spec.servers.tls.mode: SIMPLE (terminate TLS at gateway)
- spec.servers.tls.credentialName: todo-app-tls-cert (TLS certificate secret)
- spec.selector.istio: ingressgateway (select Istio ingress gateway)
- spec.servers.hosts: [todo-app.example.com, staging.todo-app.example.com] (host names)

### Service Entries for mTLS

The ServiceEntry resource defines external services accessed from within the mesh. Service entries for external services use PERMISSIVE mTLS to allow plain text communication with services outside the mesh.

**JSONPlaceholder ServiceEntry:**
- apiVersion: networking.istio.io/v1beta1
- kind: ServiceEntry
- metadata.name: jsonplaceholder
- spec.hosts: [jsonplaceholder.typicode.com]
- spec.location: MESH_EXTERNAL
- spec.resolution: DNS
- spec.ports: number 443, protocol HTTPS, name https
- spec.exports: to - version: v1

**Keycloak ServiceEntry (for cross-environment access):**
- apiVersion: networking.istio.io/v1beta1
- kind: ServiceEntry
- metadata.name: keycloak-external
- spec.hosts: [keycloak.todo-app.example.com, staging-keycloak.todo-app.example.com]
- spec.location: MESH_EXTERNAL
- spec.resolution: DNS
- spec.ports: number 443, protocol HTTPS, name https
- spec.exports: to - version: v1

### Destination Rules for mTLS

Destination rules configure traffic routing to services within the mesh, including mTLS configuration for service subsets.

**Backend Destination Rule:**
- apiVersion: networking.istio.io/v1beta1
- kind: DestinationRule
- metadata.name: backend
- spec.host: backend.default.svc.cluster.local
- spec.trafficPolicy.tls.mode: ISTIO_MUTUAL (use mTLS for mesh traffic)
- spec.trafficPolicy.loadBalancer.simple: ROUND_ROBIN
- spec.subsets: version v1

**Frontend Destination Rule:**
- apiVersion: networking.istio.io/v1beta1
- kind: DestinationRule
- metadata.name: frontend
- spec.host: frontend.default.svc.cluster.local
- spec.trafficPolicy.tls.mode: ISTIO_MUTUAL (use mTLS for mesh traffic)
- spec.trafficPolicy.loadBalancer.simple: ROUND_ROBIN
- spec.subsets: version v1

**Keycloak Destination Rule:**
- apiVersion: networking.istio.io/v1beta1
- kind: DestinationRule
- metadata.name: keycloak
- spec.host: keycloak.default.svc.cluster.local
- spec.trafficPolicy.tls.mode: ISTIO_MUTUAL (use mTLS for mesh traffic)
- spec.trafficPolicy.loadBalancer.simple: ROUND_ROBIN
- spec.subsets: version v1

### mTLS Migration Path

The recommended migration path to mTLS follows these steps to ensure zero downtime and gradual adoption.

**Phase 1: Istio Installation and PERMISSIVE Mode**
- Install Istio with default configuration
- Configure global PeerAuthentication in PERMISSIVE mode
- Deploy services and verify connectivity
- Validate that services communicate correctly with plain text

**Phase 2: Enable mTLS Per Service**
- Configure workload-level PeerAuthentication for backend service in STRICT mode
- Verify backend service accepts mTLS connections from frontend
- Configure workload-level PeerAuthentication for frontend service in STRICT mode
- Verify frontend service accepts mTLS connections from backend
- Update Keycloak service PeerAuthentication to STRICT mode

**Phase 3: Global STRICT Mode**
- Change global PeerAuthentication from PERMISSIVE to STRICT mode
- Remove workload-level STRICT PeerAuthentication resources
- Verify all services communicate with mTLS only
- Validate external service integration with PERMISSIVE mode

**Phase 4: Enforce STRICT Mode**
- Remove all PERMISSIVE mode configurations
- Update ServiceEntries for external services to use appropriate mTLS settings
- Perform full connectivity testing across all services
- Monitor mTLS metrics for certificate rotation and connection establishment

---

## Argo CD GitOps

### Overview

Argo CD implements GitOps methodology for continuous deployment, ensuring the desired state defined in the Git repository is automatically synchronized to the Kubernetes cluster. Argo CD manages application lifecycle, tracks synchronization status, and provides rollback capabilities for production deployments.

### Repository Configuration

Argo CD is configured to watch the Git repository containing application manifests and Helm charts.

**Repository Configuration:**
- apiVersion: argoproj.io/v1alpha1
- kind: Repository
- metadata.name: todo-app-repo
- metadata.namespace: argocd
- spec.type: git
- spec.url: https://github.com/your-org/modernize-todo-app (Git repository URL)
- spec.ref: HEAD (branch to watch)
- spec.ignoreDifferences: (resource customizations to ignore during sync)
  - group: apps
    kind: Application
  - group: argoproj.io
    kind: Application
- spec.insecure: false (skip TLS verification for internal Git servers)
- spec.enableLfs: true (enable large file support)
- spec.enableOCI: false (OCI support not required)

### Application Configuration

Separate Application resources are defined for each environment and each service, enabling independent lifecycle management.

#### Development Backend Application

**Application Definition:**
- apiVersion: argoproj.io/v1alpha1
- kind: Application
- metadata.name: todo-app-backend-dev
- metadata.namespace: argocd
- metadata.finalizers: (delete operation behavior)
  - resources/finalizer.argocd.argoproj.io
- spec.project: todo-app-dev (project definition)
- spec.source.repoURL: https://github.com/your-org/modernize-todo-app
- spec.source.targetRevision: main (git branch or tag)
- spec.source.path: helm/todo-app (Helm chart path)
- spec.source.helm.valueFiles: values-dev.yaml (environment-specific values)
- spec.source.helm.parameters: (Helm parameters)
  - name: image.tag
    value: dev
- spec.destination.server: https://kubernetes.default.svc (cluster endpoint)
- spec.destination.namespace: dev (target namespace)
- spec.syncPolicy.automated.prune: true (prune resources not in Git)
- spec.syncPolicy.automated.selfHeal: true (automatically heal drift)
- spec.syncPolicy.automated.allowEmpty: false (fail if no resources)
- spec.syncPolicy.syncOptions: (synchronization options)
  - CreateNamespace: true (create namespace if missing)
  - PrunePropagationPolicy: foreground (propagation policy)
  - PruneLast: true (prune final resources last)
  - Validate: true (validate manifests before apply)
  - RespectIgnoreDifferences: true (respect ignore differences)

#### Staging Backend Application

**Application Definition:**
- apiVersion: argoproj.io/v1alpha1
- kind: Application
- metadata.name: todo-app-backend-staging
- metadata.namespace: argocd
- spec.project: todo-app-staging
- spec.source.repoURL: https://github.com/your-org/modernize-todo-app
- spec.source.targetRevision: staging (staging branch or tag)
- spec.source.path: helm/todo-app
- spec.source.helm.valueFiles: values-staging.yaml
- spec.destination.server: https://kubernetes.default.svc
- spec.destination.namespace: staging
- spec.syncPolicy.automated.prune: true
- spec.syncPolicy.automated.selfHeal: true
- spec.syncPolicy.syncOptions.Validate: true
- spec.syncPolicy.syncOptions.CreateNamespace: true

#### Production Backend Application

**Application Definition:**
- apiVersion: argoproj.io/v1alpha1
- kind: Application
- metadata.name: todo-app-backend-production
- metadata.namespace: argocd
- spec.project: todo-app-production
- spec.source.repoURL: https://github.com/your-org/modernize-todo-app
- spec.source.targetRevision: production (production branch or tag)
- spec.source.path: helm/todo-app
- spec.source.helm.valueFiles: values-production.yaml
- spec.destination.server: https://kubernetes.default.svc
- spec.destination.namespace: production
- spec.syncPolicy.automated.prune: true
- spec.syncPolicy.automated.selfHeal: true
- spec.syncPolicy.syncOptions.Validate: true
- spec.syncPolicy.syncOptions.CreateNamespace: true
- spec.ignoreDifferences: (resource customizations to preserve in production)
  - group: ""
    kind: Secret
    jsonPointers:
    - .data.KEYCLOAK_CLIENT_SECRET

#### Frontend Application (Per Environment)

Frontend applications follow the same pattern as backend applications with separate resources for dev, staging, and production.

**Frontend Application Definition (Staging):**
- apiVersion: argoproj.io/v1alpha1
- kind: Application
- metadata.name: todo-app-frontend-staging
- metadata.namespace: argocd
- spec.project: todo-app-staging
- spec.source.repoURL: https://github.com/your-org/modernize-todo-app
- spec.source.targetRevision: staging
- spec.source.path: helm/todo-app
- spec.source.helm.valueFiles: values-staging.yaml
- spec.destination.server: https://kubernetes.default.svc
- spec.destination.namespace: staging
- spec.syncPolicy.automated.prune: true
- spec.syncPolicy.automated.selfHeal: true
- spec.syncPolicy.syncOptions.Validate: true
- spec.syncPolicy.syncOptions.CreateNamespace: true

#### Keycloak Application (Per Environment)

Keycloak applications follow the same pattern with separate resources for dev, staging, and production.

**Keycloak Application Definition (Staging):**
- apiVersion: argoproj.io/v1alpha1
- kind: Application
- metadata.name: todo-app-keycloak-staging
- metadata.namespace: argocd
- spec.project: todo-app-staging
- spec.source.repoURL: https://github.com/your-org/modernize-todo-app
- spec.source.targetRevision: staging
- spec.source.path: helm/keycloak (Keycloak Helm chart path)
- spec.source.helm.valueFiles: values-staging.yaml
- spec.destination.server: https://kubernetes.default.svc
- spec.destination.namespace: staging
- spec.syncPolicy.automated.prune: true
- spec.syncPolicy.automated.selfHeal: true
- spec.syncPolicy.syncOptions.Validate: true
- spec.syncPolicy.syncOptions.CreateNamespace: true

### Project Configuration

Projects group applications by environment, providing lifecycle management and synchronization policies per environment.

#### Development Project

**Project Definition:**
- apiVersion: argoproj.io/v1alpha1
- kind: AppProject
- metadata.name: todo-app-dev
- spec.sourceRepos: (allowed source repositories)
  - https://github.com/your-org/modernize-todo-app
- spec.destinations: (allowed destination clusters)
  - name: dev-cluster
    server: https://kubernetes.default.svc
    namespace: dev
- spec.description: Development environment for Todo App
- spec.syncWindows: (automatic synchronization windows)
  - kind: allow
    applications: (applications to sync)
      - todo-app-backend-dev
      - todo-app-frontend-dev
      - todo-app-keycloak-dev
    clusters: (clusters to sync to)
      - https://kubernetes.default.svc
    namespaces: (namespaces to sync to)
      - dev
    manualSync: true (manual sync for development)
- spec.clusterResourceWhitelist: (allowed cluster resources)
  - group: apps
    kind: Deployment
  - group: apps
    kind: Service
  - group: apps
    kind: Ingress
  - group: apps
    kind: ConfigMap
  - group: apps
    kind: Secret

#### Staging Project

**Project Definition:**
- apiVersion: argoproj.io/v1alpha1
- kind: AppProject
- metadata.name: todo-app-staging
- spec.sourceRepos:
  - https://github.com/your-org/modernize-todo-app
- spec.destinations:
  - name: staging-cluster
    server: https://kubernetes.default.svc
    namespace: staging
- spec.description: Staging environment for Todo App
- spec.syncWindows: (automatic synchronization windows)
  - kind: schedule
    schedule: every 15 minutes
    applications:
      - todo-app-backend-staging
      - todo-app-frontend-staging
      - todo-app-keycloak-staging
    clusters:
      - https://kubernetes.default.svc
    namespaces:
      - staging
    manualSync: false (automatic sync for staging)
  - kind: allow
    applications:
      - todo-app-backend-staging
      - todo-app-frontend-staging
    clusters:
      - https://kubernetes.default.svc
    namespaces:
      - staging
    manualSync: true (allow manual sync between scheduled syncs)
- spec.clusterResourceWhitelist: (allowed cluster resources)
  - group: apps
    kind: Deployment
  - group: apps
    kind: Service
  - group: apps
    kind: Ingress
  - group: apps
    kind: ConfigMap
  - group: apps
    kind: Secret
  - group: argoproj.io
    kind: Application

#### Production Project

**Project Definition:**
- apiVersion: argoproj.io/v1alpha1
- kind: AppProject
- metadata.name: todo-app-production
- spec.sourceRepos:
  - https://github.com/your-org/modernize-todo-app
- spec.destinations:
  - name: production-cluster
    server: https://kubernetes.default.svc
    namespace: production
- spec.description: Production environment for Todo App
- spec.syncWindows: (automatic synchronization windows)
  - kind: allow
    applications:
      - todo-app-backend-production
      - todo-app-frontend-production
      - todo-app-keycloak-production
    clusters:
      - https://kubernetes.default.svc
    namespaces:
      - production
    manualSync: true (manual sync only for production)
- spec.clusterResourceWhitelist: (allowed cluster resources)
  - group: apps
    kind: Deployment
  - group: apps
    kind: StatefulSet
  - group: apps
    kind: Service
  - group: apps
    kind: Ingress
  - group: apps
    kind: ConfigMap
  - group: apps
    kind: Secret
  - group: argoproj.io
    kind: Application
- spec.orphanedResources.warn: true (warn about resources not managed by Git)
- spec.orphanedResources.warn: (orphaned resource types to warn about)
  - kind: ConfigMap
  - kind: Secret

### Sync Policy Configuration

Sync policies define when and how applications are synchronized from Git to the cluster.

#### Automated Sync Policy

Automated sync policy is used for development and staging environments, ensuring changes are automatically applied.

**Configuration:**
- spec.syncPolicy.automated.prune: true (remove resources not in Git)
- spec.syncPolicy.automated.selfHeal: true (automatically correct drift)
- spec.syncPolicy.automated.allowEmpty: false (fail if no resources found)
- spec.syncPolicy.syncOptions: (synchronization options)
  - CreateNamespace: true (create namespace if missing)
  - PrunePropagationPolicy: foreground (propune resources in order)
  - PruneLast: true (prune final resources last)
  - Validate: true (validate manifests before apply)
  - RespectIgnoreDifferences: true (respect configured ignore differences)
  - ServerSideApply: true (use server-side apply for better performance)

#### Manual Sync Policy

Manual sync policy is used for production environment, requiring explicit synchronization approval.

**Configuration:**
- spec.syncPolicy.manual: true (disable automated sync)
- spec.syncPolicy.syncOptions: (synchronization options)
  - CreateNamespace: true (create namespace if missing)
  - PrunePropagationPolicy: foreground (prune resources in order)
  - PruneLast: true (prune final resources last)
  - Validate: true (validate manifests before apply)
  - RespectIgnoreDifferences: true (respect configured ignore differences)

### Promotion to Production

Promotion to production requires manual synchronization after changes are validated in staging environment.

**Promotion Steps:**
1. Merge changes to staging branch
2. Verify staging environment synchronization completed successfully
3. Run smoke tests against staging environment
4. Update production branch tag or merge changes
5. Create Argo CD sync request for production applications
6. Monitor synchronization progress in Argo CD UI
7. Verify production environment health after sync
8. Run smoke tests against production environment
9. Rollback if production tests fail or issues detected

**Rollback Procedure:**
1. Identify failed application in Argo CD UI
2. Click rollback button on failed application
3. Confirm rollback to previous successful revision
4. Monitor rollback progress
5. Verify application health after rollback
6. Investigate root cause of failure
7. Fix issue in Git repository
8. Re-apply promotion steps after fix

---

## Alertmanager

### Overview

Alertmanager receives alerts from Prometheus and routes them to configured receivers based on alert labels. The alerting configuration provides visibility into application health, infrastructure status, and security incidents.

### Alert Configuration

Alerts are defined in Prometheus configuration with labels for routing and classification.

#### Critical Alerts (Severity: Critical)

**ApplicationDownAlert**
- Name: ApplicationDown
- Severity: Critical
- Description: Application is not responding to health checks
- Labels: severity critical, alertname ApplicationDown, component application
- Expression: up{job=todo-app-backend,job=todo-app-frontend,job=todo-app-keycloak} == 0 for 5 minutes
- Routing: PagerDuty, Slack #alerts critical channel
- Inhibit: Inhibit less severe alerts while application is down

**PodCrashLoopingAlert**
- Name: PodCrashLooping
- Severity: Critical
- Description: Pod is crash looping and failing to start
- Labels: severity critical, alertname PodCrashLooping, component pods
- Expression: rate(kube_pod_container_status_restarts_total{namespace=todo-app}[5m]) > 5
- Routing: PagerDuty, Slack #alerts critical channel

**DatabaseConnectionErrorAlert**
- Name: DatabaseConnectionError
- Severity: Critical
- Description: Database connection errors exceeding threshold
- Labels: severity critical, alertname DatabaseConnectionError, component database
- Expression: rate(pg_stat_database_conflict{datname=todo_app}[5m]) > 10 or rate(pg_stat_database_deadlock{datname=todo_app}[5m]) > 5
- Routing: PagerDuty, Slack #alerts critical channel

**CertificateExpiringSoonAlert**
- Name: CertificateExpiringSoon
- Severity: Critical
- Description: TLS certificate expiring within 7 days
- Labels: severity critical, alertname CertificateExpiringSoon, component tls
- Expression: avg(ssl_certificate_not_after{subject_CN=~todo-app.example.com} - time() / 86400 < 7
- Routing: Slack #alerts critical channel, Email security team

**KeycloakDownAlert**
- Name: KeycloakDown
- Severity: Critical
- Description: Keycloak service is down and not responding
- Labels: severity critical, alertname KeycloakDown, component keycloak
- Expression: up{job=keycloak} == 0 for 5 minutes
- Routing: PagerDuty, Slack #alerts critical channel

#### High Severity Alerts (Severity: High)

**HighErrorRateAlert**
- Name: HighErrorRate
- Severity: High
- Description: Application error rate exceeding threshold
- Labels: severity high, alertname HighErrorRate, component application
- Expression: rate(http_requests_total{job=todo-app-backend,status=~5..}[5m]) / rate(http_requests_total{job=todo-app-backend}[5m]) > 0.05 (more than 5% errors)
- Routing: Slack #alerts channel, Email dev team
- Inhibit: Inhibit if ApplicationDownAlert is firing

**HighLatencyAlert**
- Name: HighLatency
- Severity: High
- Description: Application response time P99 exceeding threshold
- Labels: severity high, alertname HighLatency, component application
- Expression: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket{job=todo-app-backend}[5m])) > 1.0 (more than 1 second P99 latency)
- Routing: Slack #alerts channel, Email dev team
- Inhibit: Inhibit if ApplicationDownAlert is firing

**PodNotReadyAlert**
- Name: PodNotReady
- Severity: High
- Description: Pod is not in ready state
- Labels: severity high, alertname PodNotReady, component pods
- Expression: kube_pod_status_ready{namespace=todo-app} == 0 for 10 minutes
- Routing: Slack #alerts channel, Email dev team
- Inhibit: Inhibit if ApplicationDownAlert is firing

**MemoryUsageHighAlert**
- Name: MemoryUsageHigh
- Severity: High
- Description: Container memory usage exceeding threshold
- Labels: severity high, alertname MemoryUsageHigh, component pods
- Expression: container_memory_usage_bytes{namespace=todo-app} / container_spec_memory_limit_bytes{namespace=todo-app} > 0.9 (more than 90% memory usage)
- Routing: Slack #alerts channel, Email dev team

**DiskSpaceLowAlert**
- Name: DiskSpaceLow
- Severity: High
- Description: Node disk space below threshold
- Labels: severity high, alertname DiskSpaceLow, component infrastructure
- Expression: node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.1 (less than 10% disk available)
- Routing: Slack #alerts channel, Email dev team

#### Medium Severity Alerts (Severity: Medium)

**CPUUsageHighAlert**
- Name: CPUUsageHigh
- Severity: Medium
- Description: Container CPU usage exceeding threshold
- Labels: severity medium, alertname CPUUsageHigh, component pods
- Expression: rate(container_cpu_usage_seconds_total{namespace=todo-app}[5m]) / kube_pod_container_resource_limits{namespace=todo-app,resource=cpu} > 0.8 (more than 80% CPU usage)
- Routing: Slack #alerts channel

**ReplicaCountMismatchAlert**
- Name: ReplicaCountMismatch
- Severity: Medium
- Description: Deployment replica count not matching desired state
- Labels: severity medium, alertname ReplicaCountMismatch, component pods
- Expression: kube_deployment_status_replicas_available{namespace=todo-app} != kube_deployment_spec_replicas{namespace=todo-app} for 15 minutes
- Routing: Slack #alerts channel

**IngressRateLimitAlert**
- Name: IngressRateLimit
- Severity: Medium
- Description: Ingress rate limit exceeded
- Labels: severity medium, alertname IngressRateLimit, component ingress
- Expression: rate(nginx_ingress_controller_requests{status=~4..}[5m]) > 10000 (more than 10000 rate-limited requests per 5 minutes)
- Routing: Slack #alerts channel

**PodRestartAlert**
- Name: PodRestart
- Severity: Medium
- Description: Pod restarted within last hour
- Labels: severity medium, alertname PodRestart, component pods
- Expression: increase(kube_pod_container_status_restarts_total{namespace=todo-app}[1h]) > 0
- Routing: Slack #alerts channel

#### Low Severity Alerts (Severity: Low)

**DeploymentRollingUpdateAlert**
- Name: DeploymentRollingUpdate
- Severity: Low
- Description: Deployment is performing rolling update
- Labels: severity low, alertname DeploymentRollingUpdate, component deployment
- Expression: kube_deployment_status_replicas_updated{namespace=todo-app} > 0
- Routing: Slack #notifications channel

**NodeReadyAlert**
- Name: NodeReady
- Severity: Low
- Description: Node is not in ready state
- Labels: severity low, alertname NodeReady, component infrastructure
- Expression: kube_node_status_condition{condition=Ready,status=true} == 0 for 30 minutes
- Routing: Slack #notifications channel

**CertificateRenewalReminder**
- Name: CertificateRenewalReminder
- Severity: Low
- Description: TLS certificate expiring within 30 days
- Labels: severity low, alertname CertificateRenewalReminder, component tls
- Expression: avg(ssl_certificate_not_after{subject_CN=~todo-app.example.com} - time() / 86400 < 30 and > 7
- Routing: Slack #notifications channel, Email dev team

### Alert Routing

Alert routing configuration directs alerts to appropriate receivers based on severity and labels.

#### Receiver Configuration

**PagerDuty Receiver:**
- name: pagerduty
- service_key: PAGERDUTY_SERVICE_KEY
- severity: critical (only route critical severity alerts to PagerDuty)
- description: Route critical alerts to PagerDuty for on-call response

**Slack Receiver:**
- name: slack-critical
- api_url: SLACK_WEBHOOK_URL_CRITICAL
- channel: #alerts-critical
- title: Critical Alert: {{ .CommonLabels.alertname }}
- text: {{ range .Alerts }}{{ .Annotations.description }}{{ end }}
- color: danger (red for critical alerts)

**Slack Receiver (High):**
- name: slack-high
- api_url: SLACK_WEBHOOK_URL_HIGH
- channel: #alerts
- title: High Alert: {{ .CommonLabels.alertname }}
- text: {{ range .Alerts }}{{ .Annotations.description }}{{ end }}
- color: warning (orange for high alerts)

**Slack Receiver (Medium):**
- name: slack-medium
- api_url: SLACK_WEBHOOK_URL_MEDIUM
- channel: #notifications
- title: Alert: {{ .CommonLabels.alertname }}
- text: {{ range .Alerts }}{{ .Annotations.description }}{{ end }}
- color: good (yellow for medium alerts)

**Slack Receiver (Low):**
- name: slack-low
- api_url: SLACK_WEBHOOK_URL_LOW
- channel: #notifications
- title: Notification: {{ .CommonLabels.alertname }}
- text: {{ range .Alerts }}{{ .Annotations.description }}{{ end }}
- color: info (blue for low alerts)

**Email Receiver:**
- name: email-dev-team
- to: dev-team@example.com (development team email)
- from: alertmanager@example.com
- headers:
  - Subject: {{ .Status | toUpper }}: {{ .CommonLabels.alertname }}
  - X-Prometheus-Alert: {{ .CommonLabels.alertname }}
  - X-Prometheus-Severity: {{ .CommonLabels.severity }}
- html: {{ template .Content }}{{ end }}

**Email Receiver (Security):**
- name: email-security-team
- to: security-team@example.com (security team email)
  from: alertmanager@example.com
  headers:
  - Subject: SECURITY ALERT: {{ .CommonLabels.alertname }}
  - X-Prometheus-Alert: {{ .CommonLabels.alertname }}
  - X-Prometheus-Severity: {{ .CommonLabels.severity }}
  html: {{ template .Content }}{{ end }}

#### Route Configuration

**Critical Severity Route:**
- match:
    - severity: critical
  receiver: pagerduty, slack-critical, email-security-team (for security-related critical alerts)
  group_by: [alertname, component] (group alerts by name and component)
  group_wait: 30s (wait 30 seconds for group evaluation)
  group_interval: 5m (send grouped alerts every 5 minutes)
  repeat_interval: 12h (repeat every 12 hours if alert continues)
  routes:
    - match:
        - alertname: CertificateExpiringSoon
      receiver: slack-critical, email-security-team
    - continue: true
    - match:
        - severity: critical
      receiver: pagerduty, slack-critical

**High Severity Route:**
- match:
    - severity: high
  receiver: slack-high, email-dev-team
  group_by: [alertname, component]
  group_wait: 30s
  group_interval: 10m
  repeat_interval: 24h

**Medium Severity Route:**
- match:
    - severity: medium
  receiver: slack-medium
  group_by: [alertname, component]
  group_wait: 1m
  group_interval: 30m
  repeat_interval: 0 (do not repeat medium alerts)

**Low Severity Route:**
- match:
    - severity: low
  receiver: slack-low
  group_by: [alertname]
  group_wait: 5m
  group_interval: 1h
  repeat_interval: 0 (do not repeat low alerts)

### Alert Inhibition

Inhibition rules suppress less important alerts when more important alerts are firing.

**Application Down Inhibition:**
- source_match:
    - alertname: ApplicationDown
  target_match:
    - severity: high,medium,low (inhibit less severe alerts)
  equal:
    - component: application
- inhibit if: component, namespace match

**Pod Crash Looping Inhibition:**
- source_match:
    - alertname: PodCrashLooping
  target_match:
    - severity: high,medium,low
  equal:
    - component: pods
- inhibit if: component, namespace match

**Keycloak Down Inhibition:**
- source_match:
    - alertname: KeycloakDown
  target_match:
    - alertname: ApplicationDown (backend may show as down when Keycloak is down)
  equal:
    - component: backend
- inhibit if: namespace match

**Memory High Inhibition:**
- source_match:
    - alertname: MemoryUsageHigh
  target_match:
    - alertname: ApplicationDown (application may be down due to OOM)
  equal:
    - component: application
- inhibit if: component, namespace match

### Alert Silencing

Silence rules temporarily suppress alerts during planned maintenance or known transient conditions.

**Deployment Window Silence:**
- matchers:
    - name: DeploymentMaintenance
      regex: DeploymentRollingUpdate (suppress deployment rolling update alerts during planned deployments)
  startAt: 2026-03-09T00:00:00-05:00 (start time in UTC)
  endAt: 2026-03-09T00:30:00-05:00 (end time in UTC)
  createdBy: deployment-bot

**Testing Environment Silence:**
- matchers:
    - namespace: dev (suppress non-critical alerts in dev environment)
    - severity: high,medium,low
- comments: Development environment expected to have transient issues

**Certificate Renewal Maintenance Silence:**
- matchers:
    - alertname: CertificateRenewalReminder
  startAt: 2026-03-09T00:00:00-05:00
  endAt: 2026-03-09T01:00:00-05:00
  createdBy: security-bot

---

## Appendix

### Terraform State Management

Terraform state files are managed with S3 backend for state locking and remote state storage. Each environment has a dedicated state file and S3 bucket prefix to prevent cross-environment state conflicts.

**S3 Backend Configuration:**
- bucket: todo-app-terraform-state
- key: terraform.tfstate (state file path)
- region: us-east-1 (same region as infrastructure)
- encrypt: true (enable state encryption at rest)
- acl: bucket-owner-full-control (S3 ACL for ownership)
- dynamodb_table: terraform-state-lock (DynamoDB table for state locking)
- dynamodb_region: us-east-1 (DynamoDB region for lock table)

### Helm Chart Lifecycle

Helm chart versioning follows semantic versioning with major, minor, and patch versions. Chart upgrades are tested in staging before promotion to production.

**Versioning:**
- Major version (X.Y.Z): Breaking changes requiring manual intervention
- Minor version (X.Y.Z): New features backward compatible
- Patch version (X.Y.Z): Bug fixes and documentation updates

**Upgrade Strategy:**
- Staging: Always upgrade to latest chart version
- Production: Upgrade to specific tested version from staging

### Argo CD Synchronization Strategies

Argo CD synchronization strategies balance automation with control for different environments.

**Development:**
- Strategy: Automated sync with 15-minute interval
- Self-Heal: Enabled automatically correct drift
- Manual Sync: Available on-demand for testing
- Rollback: Available via Argo CD UI

**Staging:**
- Strategy: Scheduled sync with 15-minute interval
- Self-Heal: Enabled automatically correct drift
- Manual Sync: Available on-demand between scheduled syncs
- Rollback: Available via Argo CD UI with rollback history

**Production:**
- Strategy: Manual sync only (no automated sync)
- Self-Heal: Enabled for automatic drift correction
- Manual Sync: Required for all changes
- Rollback: Available via Argo CD UI with rollback history and approval required

### Alert Severity Escalation

Alert severity escalation paths define how alerts are handled based on duration and impact.

**Critical Escalation:**
- Immediate: PagerDuty notification, Slack critical channel
- 5 minutes: Security team notification (if security-related)
- 15 minutes: Engineering management notification
- 30 minutes: Incident declaration if not resolved

**High Escalation:**
- Immediate: Slack alerts channel, email dev team
- 15 minutes: Engineering lead notification
- 30 minutes: Incident declaration if not resolved

**Medium Escalation:**
- Immediate: Slack notifications channel
- 30 minutes: Engineering lead notification if not resolved
- 1 hour: Team lead notification if not resolved

**Low Escalation:**
- Immediate: Slack notifications channel
- 4 hours: Engineering lead notification if not resolved
- 8 hours: Team lead notification if not resolved

---

**Document Version:** 1.0
**Last Updated:** 2026-03-09
