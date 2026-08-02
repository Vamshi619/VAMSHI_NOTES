# EC2 + Jenkins Curriculum for AI Engineering, MLOps and LLMOps

This curriculum treats **EC2 as the compute infrastructure** and **Jenkins as the automation engine** for building, testing, training, evaluating and deploying AI systems.

A production-oriented architecture typically looks like:

```text
Developer
   |
   v
GitHub / GitLab / Bitbucket
   |
   | Webhook
   v
Jenkins Controller on EC2
   |
   +---- CPU Jenkins Agent
   |
   +---- GPU Jenkins Agent
   |
   +---- Docker Build Agent
   |
   v
Tests / Data Validation / Model Evaluation / LLM Evaluation
   |
   v
Docker Image
   |
   v
Amazon ECR
   |
   v
EC2 / Auto Scaling Group / ECS / EKS
   |
   v
Application Load Balancer
   |
   v
CloudWatch + Model/LLM Monitoring
```

Amazon EC2 provides scalable virtual computing, including general-purpose, memory-optimized, GPU, Trainium and Inferentia-based instance families. AWS also provides Deep Learning AMIs containing common deep-learning frameworks and GPU dependencies. ([AWS Documentation][1])

---

# 1. Prerequisites

Before starting EC2 and Jenkins, learn the following foundations.

## 1.1 Linux

You must be comfortable working without a graphical interface.

Topics:

* Linux directory structure
* Files and directories
* Users, groups and permissions
* `chmod`, `chown`, `sudo`
* Processes and signals
* `ps`, `top`, `htop`, `kill`
* Services using `systemctl`
* Logs using `journalctl`
* Package managers:

  * `apt`
  * `yum`
  * `dnf`
* Environment variables
* Shell profiles
* Disk usage:

  * `df`
  * `du`
  * `lsblk`
  * `mount`
* Network commands:

  * `curl`
  * `wget`
  * `ping`
  * `telnet`
  * `nc`
  * `ss`
  * `netstat`
* Text-processing commands:

  * `grep`
  * `awk`
  * `sed`
  * `cut`
  * `sort`
  * `uniq`
  * `jq`
* Compression:

  * `tar`
  * `gzip`
  * `zip`
* Secure copy:

  * `scp`
  * `rsync`
* SSH configuration
* Bash scripting
* Exit codes
* Pipes and redirection
* Cron jobs
* Log rotation

### Required practical exercise

Create a Linux service using `systemd` for a Python FastAPI application and verify:

```bash
systemctl status my-ai-service
journalctl -u my-ai-service -f
curl http://localhost:8000/health
```

---

## 1.2 Networking

Learn:

* IPv4
* Public and private IP addresses
* CIDR notation
* Subnets
* Route tables
* Internet Gateway
* NAT Gateway
* DNS
* Ports
* TCP versus UDP
* HTTP and HTTPS
* TLS certificates
* Reverse proxies
* Load balancers
* Firewalls
* Stateful versus stateless filtering
* Proxy servers
* Webhooks

You should understand what happens when a user calls:

```text
https://api.example.com/predict
```

The expected flow is:

```text
DNS
→ Load Balancer
→ Security Group
→ EC2 Network Interface
→ Nginx
→ Python API
→ Model
→ Response
```

---

## 1.3 Git

Learn:

* Repository creation
* Clone, pull, add, commit and push
* Branches
* Merge and rebase
* Tags
* Pull requests
* Merge conflicts
* `.gitignore`
* Git credentials
* SSH authentication
* Personal access tokens
* Webhooks
* Branch protection
* Release tags
* Semantic versioning

Jenkins must react to source-code events, so Git knowledge is mandatory.

---

## 1.4 Python and API fundamentals

Learn:

* Python virtual environments
* Dependency management
* `requirements.txt`
* `pyproject.toml`
* Packaging
* Unit testing with `pytest`
* Type checking
* Linting
* FastAPI or Flask
* Environment-based configuration
* Logging
* Health endpoints
* API timeouts
* Exception handling
* Graceful shutdown

---

## 1.5 Docker

Learn:

* Images and containers
* Dockerfile
* Layers
* Build context
* Image tags
* Volumes
* Networks
* Environment variables
* Container logs
* Health checks
* Multi-stage builds
* Docker Compose
* Registry authentication
* CPU and memory limits
* GPU containers
* NVIDIA Container Toolkit
* Image vulnerability scanning

---

# 2. Amazon EC2 Curriculum

## Module 1: EC2 fundamentals

Learn the core EC2 concepts:

* EC2 instance
* Amazon Machine Image
* Instance type
* Region
* Availability Zone
* Key pair
* Security group
* Elastic Network Interface
* Public IPv4 address
* Private IPv4 address
* Elastic IP
* Instance profile
* User data
* Tags
* Root volume
* Instance lifecycle

Instance lifecycle states:

```text
pending
→ running
→ stopping
→ stopped
→ shutting-down
→ terminated
```

Understand the difference between:

* Reboot
* Stop
* Start
* Hibernate
* Terminate

### Practical exercise

Launch an Ubuntu or Amazon Linux instance and:

1. Connect using SSH.
2. Install Python.
3. Run a FastAPI application.
4. Open only the required application port.
5. Test the application from your local machine.
6. Stop and restart the instance.
7. Verify whether its public IP changed.

---

## Module 2: EC2 instance types

Understand how to select compute according to workload.

### General categories

| Category              | Typical use                                                |
| --------------------- | ---------------------------------------------------------- |
| General purpose       | Jenkins controller, API server, small inference workloads  |
| Compute optimized     | Data processing, CPU model inference, builds               |
| Memory optimized      | Large feature processing, vector indexes, in-memory models |
| Storage optimized     | High-throughput local data processing                      |
| Accelerated computing | GPU training, GPU inference, LLM hosting                   |
| HPC optimized         | Distributed training and tightly coupled workloads         |

### AI-specific selection criteria

Evaluate:

* Number of vCPUs
* RAM
* GPU model
* GPU count
* GPU memory
* Network bandwidth
* EBS bandwidth
* Local NVMe storage
* CPU architecture
* Availability in the selected Region
* Framework compatibility
* Driver compatibility
* Cost per training run
* Cost per inference request

### AI accelerator topics

Learn the purpose of:

* NVIDIA GPU instance families
* AWS Inferentia for inference
* AWS Trainium for training
* Graviton processors
* CPU-only inference
* Multi-GPU instances
* Multi-node GPU clusters

AWS documents accelerated families covering GPU, Inferentia and Trainium-based instances. Instance availability and specifications vary by family and Region, so selection should be based on current EC2 instance specifications rather than memorizing one instance name. ([AWS Documentation][2])

---

## Module 3: AMIs and instance bootstrapping

Learn:

* Public AMIs
* AWS-provided AMIs
* Marketplace AMIs
* Custom AMIs
* AMI versioning
* AMI ownership
* AMI permissions
* AMI copying across Regions
* Golden images
* Image deprecation
* Root-volume snapshots
* Building immutable servers

### User data and cloud-init

Use user data to:

* Install Jenkins
* Install Docker
* Install the CloudWatch agent
* Configure environment variables
* Download application files
* Register an instance as a Jenkins agent
* Start application services

EC2 user data can run configuration scripts during instance launch and is commonly used to customize instances created from reusable AMIs. ([AWS Documentation][3])

### AI-specific AMIs

Study:

* Deep Learning AMI
* CUDA versions
* cuDNN
* NVIDIA drivers
* PyTorch environment
* TensorFlow environment
* Conda environments
* Base GPU AMI versus framework-specific AMI
* Building a reusable GPU image

AWS Deep Learning AMIs provide preconfigured environments for CPU and GPU-based deep-learning workloads. ([AWS Documentation][4])

---

## Module 4: EC2 storage

### EBS

Learn:

* EBS volumes
* Root and additional volumes
* Volume types
* Size, IOPS and throughput
* Attach and detach
* Mounting
* File systems
* Resizing
* Encryption
* Snapshots
* Snapshot restoration
* Cross-Region snapshot copying
* Delete-on-termination
* Backup policies

EBS provides persistent block storage for EC2 instances and supports volume modification and snapshots. ([AWS Documentation][5])

### Instance store

Learn:

* Temporary local storage
* High-speed NVMe
* Data loss when an instance is terminated
* Appropriate use for caches, temporary checkpoints and intermediate files

### Storage selection for AI workloads

| Requirement                                | Common option                              |
| ------------------------------------------ | ------------------------------------------ |
| Jenkins home directory                     | EBS                                        |
| Source code checkout                       | EBS                                        |
| Docker build cache                         | EBS or instance store                      |
| Training datasets                          | S3                                         |
| Model artifacts                            | S3                                         |
| Shared Linux filesystem                    | EFS                                        |
| High-performance distributed training data | FSx for Lustre                             |
| Temporary preprocessing                    | Instance store                             |
| Vector database data                       | EBS, managed database or dedicated storage |

FSx for Lustre can be accessed from EC2 and is designed for high-performance workloads such as machine learning and HPC. ([AWS Documentation][6])

---

## Module 5: EC2 networking and VPC

Learn:

* Default VPC versus custom VPC
* Public subnet
* Private subnet
* Route tables
* Internet Gateway
* NAT Gateway
* VPC endpoints
* Network ACLs
* Security groups
* Elastic Network Interfaces
* Elastic IP addresses
* DNS hostnames
* VPC peering
* Transit Gateway basics
* PrivateLink basics
* VPC Flow Logs

Security groups operate as instance-level virtual firewalls controlling inbound and outbound traffic. VPC configuration determines IP ranges, subnets, routing, gateways and network isolation. ([AWS Documentation][7])

### Recommended Jenkins network design

```text
Public Subnet
    |
Application Load Balancer or Reverse Proxy
    |
Private Subnet
    |
Jenkins Controller EC2
    |
Private Jenkins Agents
```

Avoid exposing Jenkins directly to the internet.

### Ports you should understand

| Port                 | Use                                            |
| -------------------- | ---------------------------------------------- |
| 22                   | SSH                                            |
| 80                   | HTTP                                           |
| 443                  | HTTPS                                          |
| 8080                 | Default Jenkins web interface                  |
| 50000                | Jenkins inbound agent communication, when used |
| Application-specific | FastAPI, model server or monitoring endpoints  |

Do not open Jenkins ports to `0.0.0.0/0` unless there is a tightly controlled reason.

---

## Module 6: IAM for EC2 and Jenkins

Learn:

* IAM users
* IAM groups
* IAM policies
* IAM roles
* Instance profiles
* Service roles
* Trust policies
* Permission policies
* Resource-based policies
* Least privilege
* Temporary credentials
* STS
* `iam:PassRole`
* Cross-account role assumption
* AWS Organizations basics
* Permission boundaries
* Access Analyzer

### Important rule

Do not store permanent AWS access keys inside:

* Jenkinsfile
* Git repository
* Dockerfile
* Shell scripts
* `.env` committed to Git
* Jenkins console output

Attach an IAM role to the Jenkins controller or, preferably, separate roles to individual Jenkins agents. EC2 applications can use temporary credentials made available through the instance metadata service, and AWS SDKs and the CLI can retrieve these automatically. ([AWS Documentation][8])

### Suggested role separation

```text
JenkinsControllerRole
    - Read configuration
    - Discover agents
    - Limited administration

JenkinsBuildAgentRole
    - Pull source dependencies
    - Push images to ECR
    - Upload build artifacts to S3

JenkinsDeploymentAgentRole
    - Deploy application
    - Update Auto Scaling Group
    - Invoke CodeDeploy

JenkinsGPUTrainingRole
    - Read training data
    - Write checkpoints
    - Write model artifacts
```

---

## Module 7: EC2 access and Systems Manager

Learn:

* SSH key pairs
* Private-key permissions
* Bastion hosts
* Session Manager
* Run Command
* Patch Manager
* Inventory
* Parameter Store
* State Manager
* Fleet Manager

AWS Systems Manager can manage and access EC2 instances when the required role and agent configuration are present. ([AWS Documentation][9])

### Recommended progression

1. Learn SSH for troubleshooting.
2. Learn Session Manager.
3. Prefer private instances without public SSH exposure.
4. Use Run Command for repeatable fleet operations.
5. Use Patch Manager for operating-system updates.

---

## Module 8: EC2 security hardening

Learn:

* Least-privilege security groups
* Private subnets
* EBS encryption
* KMS keys
* TLS
* Reverse proxies
* OS patching
* Package updates
* Antivirus or endpoint controls where required
* Disable password-based SSH
* Rotate keys
* Audit login activity
* Restrict sudo access
* Secrets Manager
* Parameter Store
* CloudTrail
* GuardDuty
* AWS Config
* Security Hub basics
* Vulnerability scanning
* Image hardening
* CIS benchmark awareness

### Instance Metadata Service

Understand:

* IMDS
* IMDSv1
* IMDSv2
* Session tokens
* Metadata hop limit
* Risks of server-side request forgery
* Enforcing IMDSv2

AWS supports requiring IMDSv2 when launching or modifying instances. Existing applications must support IMDSv2 before IMDSv1 is disabled. ([AWS Documentation][10])

---

## Module 9: Load balancing and Auto Scaling

Learn:

* Application Load Balancer
* Network Load Balancer
* Target groups
* Health checks
* Listener rules
* Path-based routing
* Host-based routing
* TLS termination
* Sticky sessions
* Auto Scaling Groups
* Launch templates
* Minimum, desired and maximum capacity
* Scaling policies
* Scheduled scaling
* Target-tracking scaling
* Instance refresh
* Lifecycle hooks
* Warm pools
* Health-check grace period
* Multi-AZ deployment

Auto Scaling Groups can integrate with Elastic Load Balancing so that instances are registered and traffic is distributed across healthy targets. ([AWS Documentation][11])

### AI inference scaling metrics

Do not rely only on CPU.

Consider:

* Requests per second
* Active requests
* Queue depth
* Model latency
* GPU utilization
* GPU memory usage
* Tokens generated per second
* Concurrent model sessions
* Batch size
* Request timeout rate

---

## Module 10: EC2 purchasing and cost control

Learn:

* On-Demand Instances
* Spot Instances
* Savings Plans
* Reserved Instances
* Dedicated Hosts
* Capacity Reservations
* Capacity Blocks for ML
* Data-transfer charges
* EBS charges
* Snapshot charges
* Elastic IP charges
* Idle-resource identification
* Cost allocation tags
* Budgets and alerts

### Spot for AI workloads

Good candidates:

* Interruptible model training
* Batch inference
* Embedding generation
* Data preprocessing
* Hyperparameter tuning
* Temporary Jenkins build agents
* Non-production testing

Poor candidates without additional protection:

* Jenkins controller
* Stateful databases
* Uncheckpointed long-running training
* Critical synchronous inference

Spot instances use spare EC2 capacity and can be interrupted. AWS recommends interruption-aware designs and flexible instance and Availability Zone selection. ([AWS Documentation][12])

Learn:

* Spot interruption notices
* Checkpointing
* Resume logic
* Mixed-instance Auto Scaling Groups
* Capacity-optimized allocation
* Graceful agent shutdown
* Jenkins retry handling

For guaranteed GPU availability during planned training periods, understand Capacity Reservations and Capacity Blocks for ML. ([AWS Documentation][13])

---

## Module 11: Monitoring and observability

Learn EC2 monitoring using:

* CloudWatch metrics
* CloudWatch Logs
* CloudWatch alarms
* CloudWatch dashboards
* EventBridge
* SNS notifications
* CloudTrail
* VPC Flow Logs
* Application logs
* System metrics
* Custom metrics

### Metrics to monitor

#### EC2

* CPU utilization
* Network input/output
* Disk operations
* Status-check failures
* Memory usage
* Disk usage
* Process health

#### GPU

* GPU utilization
* GPU memory
* GPU temperature
* Power consumption
* CUDA errors
* NVIDIA driver failures

#### Jenkins

* Queue length
* Executor utilization
* Agent availability
* Build duration
* Build failure rate
* Controller memory
* JVM heap
* Disk usage
* Plugin failures

The CloudWatch agent can collect host-level metrics, logs and traces from EC2 and other environments. ([AWS Documentation][14])

---

## Module 12: Infrastructure as Code

Learn at least one primary tool:

* Terraform
* AWS CloudFormation
* AWS CDK

Supporting tools:

* Packer for AMIs
* Ansible for configuration
* Shell scripting
* Jenkins Configuration as Code

Infrastructure resources that should eventually be automated:

* VPC
* Subnets
* Security groups
* IAM roles
* EC2 launch templates
* Auto Scaling Groups
* Load balancers
* EBS volumes
* ECR repositories
* S3 buckets
* CloudWatch alarms
* Jenkins controller
* Jenkins agents

---

## Module 13: Advanced AI compute on EC2

Important for senior MLOps engineers and large-model workloads.

Learn:

* Multi-GPU training
* Distributed data parallel training
* Data parallelism
* Tensor parallelism
* Pipeline parallelism
* Model sharding
* NCCL
* CUDA
* cuDNN
* GPU topology
* Cluster placement groups
* Elastic Fabric Adapter
* Checkpoint storage
* Shared high-throughput filesystems
* GPU capacity planning
* GPU failure recovery
* GPU scheduling
* Training-job isolation

EFA provides lower-latency, higher-throughput inter-instance communication for AI, ML and HPC workloads. Cluster placement groups place tightly coupled instances close together for low-latency communication. ([AWS Documentation][15])

---

# 3. Jenkins Curriculum

## Module 1: Jenkins fundamentals

Learn:

* Continuous Integration
* Continuous Delivery
* Continuous Deployment
* Jenkins controller
* Jenkins agent
* Node
* Executor
* Workspace
* Job
* Build
* Queue
* Artifact
* Plugin
* Credential
* Pipeline
* Stage
* Step

### Architecture

```text
Jenkins Controller
    |
    +-- Stores job configuration
    +-- Schedules builds
    +-- Maintains build history
    +-- Coordinates agents
    |
    +---- Linux CPU Agent
    +---- Docker Agent
    +---- GPU Agent
    +---- Deployment Agent
```

Do not run every build directly on the controller.

---

## Module 2: Installing Jenkins on EC2

Learn:

* Jenkins prerequisites
* Java installation
* Package installation
* Jenkins repository configuration
* Initial administrator setup
* Unlocking Jenkins
* Jenkins home directory
* Service management
* JVM settings
* Reverse-proxy setup
* HTTPS
* Domain and DNS
* Persistent EBS storage
* Backup preparation
* Upgrade preparation

### Practical installation architecture

```text
EC2
├── Jenkins service
├── Java
├── /var/lib/jenkins on EBS
├── Nginx
├── TLS certificate
└── CloudWatch agent
```

### Commands you should understand

```bash
sudo systemctl start jenkins
sudo systemctl stop jenkins
sudo systemctl restart jenkins
sudo systemctl status jenkins
sudo journalctl -u jenkins -f
```

---

## Module 3: Jenkins job types

Learn:

* Freestyle project
* Pipeline project
* Multibranch Pipeline
* Folder
* Organization Folder
* Parameterized job

For professional work, focus mainly on:

```text
Pipeline
Multibranch Pipeline
Jenkinsfile
Shared Libraries
```

---

## Module 4: Pipeline as Code

Jenkins Pipeline supports defining delivery workflows in a `Jenkinsfile`, including branch workflows, Docker-based stages and shared libraries. ([Jenkins][16])

Learn both:

### Declarative Pipeline

```groovy
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh 'pytest'
            }
        }
    }
}
```

### Scripted Pipeline

```groovy
node {
    stage('Test') {
        sh 'pytest'
    }
}
```

Use Declarative Pipeline for most projects. Learn Scripted Pipeline sufficiently to understand existing pipelines and advanced logic.

### Essential Jenkinsfile topics

* `pipeline`
* `agent`
* `stages`
* `stage`
* `steps`
* `environment`
* `parameters`
* `tools`
* `options`
* `when`
* `input`
* `post`
* `script`
* `parallel`
* `matrix`
* `timeout`
* `retry`
* `timestamps`
* `buildDiscarder`
* `disableConcurrentBuilds`
* `stash`
* `unstash`
* `archiveArtifacts`
* `junit`
* `withCredentials`
* `catchError`

---

## Module 5: Git integration

Learn:

* Git plugin
* GitHub integration
* GitLab integration
* Bitbucket integration
* Webhooks
* Polling
* Multibranch Pipelines
* Pull-request builds
* Branch discovery
* Tag builds
* Commit-status reporting
* Branch-specific deployments

### Expected workflow

```text
Developer pushes code
→ Git webhook calls Jenkins
→ Jenkins discovers branch
→ Jenkins reads Jenkinsfile
→ Pipeline starts
→ Status is returned to Git
```

### Branch strategy example

| Branch         | Action                             |
| -------------- | ---------------------------------- |
| Feature branch | Unit tests and linting             |
| Pull request   | Full test suite and security scan  |
| Develop        | Deploy to development              |
| Release        | Deploy to staging                  |
| Main           | Production approval and deployment |
| Git tag        | Versioned production release       |

---

## Module 6: Jenkins agents

Learn:

* Static agents
* Dynamic agents
* SSH agents
* Inbound agents
* Docker agents
* EC2-based agents
* Kubernetes agents
* Agent labels
* Executors
* Workspaces
* Tool installation
* Agent isolation
* Ephemeral agents
* Agent cleanup

The Pipeline `agent` directive controls where a Pipeline or individual stage runs. ([Jenkins][17])

### Recommended labels

```text
linux
python
docker
gpu
cuda
deployment
terraform
security-scan
```

### Example

```groovy
stage('Train Model') {
    agent {
        label 'gpu'
    }

    steps {
        sh 'python train.py'
    }
}
```

### AI-specific agent design

| Agent            | Purpose                     |
| ---------------- | --------------------------- |
| CPU agent        | Tests, linting, packaging   |
| Docker agent     | Image building              |
| GPU agent        | Training and GPU validation |
| Deployment agent | Production deployment       |
| Terraform agent  | Infrastructure changes      |
| Evaluation agent | Model or LLM evaluation     |

---

## Module 7: Jenkins credentials

Learn:

* Secret text
* Username and password
* SSH private key
* Secret files
* Certificates
* Credential IDs
* Global versus folder scope
* Stage-level credential scope
* Credentials Binding
* Credential masking
* Rotation
* External secret stores

Jenkins stores configured credentials in encrypted form and exposes them to Pipelines through credential IDs. Masking reduces accidental exposure, but an untrusted Pipeline can still misuse credentials. ([Jenkins][18])

### AWS best practice

Prefer:

```text
EC2 IAM Role
```

over:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

### Other secrets

Store through:

* AWS Secrets Manager
* Systems Manager Parameter Store
* Vault
* Jenkins Credentials, where appropriate

---

## Module 8: Build and test automation

### Python pipeline

Learn to automate:

```text
Install dependencies
→ Lint
→ Type check
→ Unit tests
→ Integration tests
→ Coverage
→ Package
→ Archive results
```

Example tools:

* `pytest`
* `coverage`
* `ruff`
* `flake8`
* `black`
* `mypy`
* `bandit`
* `pip-audit`

### Quality gates

Fail the pipeline when:

* Tests fail
* Coverage is below threshold
* Critical vulnerability is found
* Model metric falls below threshold
* LLM regression exceeds threshold
* Docker image build fails
* Deployment health check fails

---

## Module 9: Docker and Jenkins

Learn:

* Running Jenkins stages in containers
* Building Docker images
* Multi-stage Dockerfiles
* Image tagging
* Immutable tags
* Commit-SHA tags
* Registry login
* ECR push
* Image scanning
* Build cache
* Cleanup
* Docker-in-Docker risks
* Host Docker socket risks

### Recommended image tags

```text
ai-service:1.4.0
ai-service:git-a82f91c
ai-service:staging
```

Avoid depending only on:

```text
latest
```

### Typical pipeline

```text
Checkout
→ Test
→ Build image
→ Scan image
→ Authenticate to ECR
→ Push image
→ Deploy image
```

---

## Module 10: Artifact management

Understand the difference between:

* Source artifact
* Build artifact
* Test report
* Docker image
* Dataset version
* Feature definition
* Model artifact
* Tokenizer
* Prompt template
* Evaluation dataset
* Vector index
* Deployment manifest

Possible storage:

| Artifact             | Storage                        |
| -------------------- | ------------------------------ |
| Python package       | Artifact repository            |
| Docker image         | ECR                            |
| Model file           | S3 or model registry           |
| Test report          | Jenkins and object storage     |
| Dataset manifest     | S3 and metadata store          |
| Evaluation results   | Database or S3                 |
| Prompt configuration | Git or prompt registry         |
| Deployment package   | S3, ECR or artifact repository |

---

## Module 11: Jenkins Shared Libraries

Learn:

* `vars/`
* `src/`
* `resources/`
* Global libraries
* Folder-scoped libraries
* Versioned libraries
* Reusable stages
* Common deployment logic
* Standard security checks
* Common notifications
* Standard model evaluation

Example reusable functions:

```groovy
pythonQualityChecks()
buildDockerImage()
scanContainer()
pushToECR()
deployToEC2()
runModelEvaluation()
runLLMEvaluation()
```

Do not create overly complex shared libraries before pipelines have clear repeated patterns.

---

## Module 12: Jenkins security

Learn:

* Authentication
* Authorization
* Matrix authorization
* Role-based authorization
* SSO
* LDAP
* SAML or OIDC
* CSRF protection
* Agent-to-controller security
* Script approval
* Plugin security advisories
* Credential scope
* Folder permissions
* Audit logging
* HTTPS
* Session management
* Restricting anonymous access
* Securing webhook endpoints
* Protecting multibranch builds from untrusted forks

Jenkins warns that pull requests from untrusted forks can access credentials if trusted secrets are exposed to code controlled by the pull request. ([Jenkins][19])

---

## Module 13: Jenkins administration

Learn:

* Plugin installation
* Plugin dependencies
* Plugin upgrades
* Jenkins upgrades
* LTS releases
* Upgrade testing
* JVM heap
* Garbage collection
* Disk management
* Build retention
* Log retention
* Workspace cleanup
* Node management
* Controller monitoring
* System logs
* Troubleshooting
* Reverse proxy
* Email and chat notifications

Jenkins administration includes managing plugins, nodes, tools and system configuration. ([Jenkins][20])

---

## Module 14: Jenkins Configuration as Code

Learn:

* JCasC YAML
* Controller configuration
* Credentials references
* Plugin configuration
* Agents
* Authorization
* Global tools
* Reproducible controller setup
* Configuration validation
* Git versioning
* Environment-specific overlays

Jenkins Configuration as Code represents controller configuration using human-readable YAML that can be stored and managed as source code. ([Jenkins][21])

---

## Module 15: Backup and disaster recovery

Learn what is stored in `JENKINS_HOME`:

* Job configurations
* Build history
* Plugins
* Credentials metadata
* User configuration
* Pipeline state
* Node configuration

Backup topics:

* EBS snapshots
* Filesystem backup
* Configuration backup
* Plugin inventory
* Encryption-key protection
* Off-instance backup
* Cross-Region backup
* Restore testing
* Recovery-time objective
* Recovery-point objective

Jenkins documentation provides backup and restoration guidance and notes that backup plugins are available, although operational teams should still design and test a complete restore procedure. ([Jenkins][22])

---

## Module 16: Jenkins scaling and monitoring

Learn:

* Controller bottlenecks
* Queue growth
* Executor saturation
* Agent provisioning
* Large build histories
* Large logs
* Plugin overhead
* JVM memory
* Disk I/O
* Network limitations
* Controller versus agent responsibilities
* Multiple controllers
* Folder-level separation
* Dynamic agent provisioning

Jenkins documents scaling for large numbers of users, projects, agents, jobs and concurrent workloads. ([Jenkins][23])

Monitoring tools:

* Prometheus
* Grafana
* CloudWatch
* OpenTelemetry
* Jenkins metrics plugins
* Log aggregation

Jenkins provides documented monitoring integrations including Prometheus and Grafana. ([Jenkins][24])

---

# 4. AI Engineering CI/CD Curriculum

An AI engineer primarily deploys AI-enabled applications rather than managing the complete statistical model lifecycle.

## Pipeline stages

```text
Checkout
→ Install dependencies
→ Lint
→ Unit test
→ Test external integrations
→ Test model/API client
→ Build Docker image
→ Security scan
→ Push to ECR
→ Deploy to EC2
→ Health check
→ Smoke test
→ Promote
```

## Topics

* FastAPI deployment
* Async APIs
* Background jobs
* Model API integrations
* Embedding model integration
* Vector database connectivity
* API retries and timeouts
* Circuit breakers
* Environment configuration
* Secrets
* Docker
* ECR
* EC2 deployment
* Application health endpoints
* Horizontal scaling
* Structured logging
* Request tracing
* Cost and latency monitoring

---

# 5. MLOps Curriculum Using EC2 and Jenkins

## Module 1: Data pipeline automation

Learn:

* Dataset ingestion
* Schema validation
* Missing-value checks
* Duplicate detection
* Range validation
* Data-type validation
* Feature validation
* Data drift
* Dataset versioning
* Data lineage
* Train-validation-test split
* Leakage detection
* Data-quality reports

Pipeline:

```text
Fetch data
→ Validate schema
→ Validate quality
→ Generate dataset version
→ Store dataset metadata
→ Approve for training
```

---

## Module 2: Training pipeline

Learn:

* Reproducible training
* Configuration-driven experiments
* Random seeds
* Hyperparameters
* CPU versus GPU training
* Checkpoints
* Distributed training
* Spot interruption recovery
* Training logs
* Experiment tracking
* Model artifact storage
* Training metadata

Pipeline:

```text
Validated dataset
→ Feature engineering
→ Train
→ Save checkpoint
→ Evaluate
→ Register model
```

---

## Module 3: Model evaluation

Learn:

* Accuracy
* Precision
* Recall
* F1 score
* ROC-AUC
* RMSE
* MAE
* MAPE
* Business metrics
* Segment-level performance
* Bias and fairness checks
* Baseline comparison
* Champion-challenger comparison
* Regression thresholds

Example quality gate:

```groovy
if (f1Score < 0.85) {
    error('Model quality gate failed')
}
```

Do not promote models based only on successful code execution.

---

## Module 4: Model registry

Learn:

* Model version
* Model stage
* Experiment ID
* Dataset version
* Git commit
* Training parameters
* Metrics
* Model signature
* Framework version
* Approval status
* Owner
* Deployment history

Possible integrations:

* MLflow Model Registry
* SageMaker Model Registry
* Custom S3 and database registry

---

## Module 5: Model packaging

Learn:

* Pickle risks
* `joblib`
* ONNX
* TorchScript
* SavedModel
* Tokenizers
* Custom preprocessing
* Dependency locking
* Model server packaging
* Containerization
* Hardware compatibility
* CPU and GPU image variants

---

## Module 6: Model deployment patterns

Learn:

* Batch inference
* Online inference
* Asynchronous inference
* Streaming inference
* Shadow deployment
* Canary deployment
* Blue-green deployment
* A/B testing
* Rolling deployment
* Rollback
* Model warm-up
* Health probes
* Readiness probes
* Graceful termination

AWS CodeDeploy can automate deployments to EC2 and supports deployment revisions containing source files, scripts and an AppSpec file. ([AWS Documentation][25])

---

## Module 7: Model monitoring

Monitor:

* Input-data drift
* Prediction drift
* Concept drift
* Accuracy decay
* Missing features
* Invalid feature values
* Prediction latency
* Throughput
* Error rate
* Model memory
* CPU/GPU utilization
* Business KPI changes
* Distribution changes by customer segment

Create retraining triggers carefully. Not every drift alert should automatically deploy a new model.

---

# 6. LLMOps Curriculum Using EC2 and Jenkins

## Module 1: LLM application versioning

Version:

* Application code
* System prompt
* User prompt templates
* Few-shot examples
* Model provider
* Model name
* Model parameters
* Temperature
* Maximum tokens
* Tool definitions
* Guardrails
* Knowledge-base version
* Embedding model
* Chunking configuration
* Retrieval parameters
* Evaluation dataset

A production response should be traceable to:

```text
Git commit
+ prompt version
+ model version
+ knowledge-base version
+ configuration version
```

---

## Module 2: Prompt pipeline

Pipeline:

```text
Prompt change
→ Syntax validation
→ Variable validation
→ Golden-dataset evaluation
→ Safety evaluation
→ Regression comparison
→ Human approval
→ Deploy configuration
```

Validate:

* Required variables
* Unused variables
* Missing placeholders
* Output schema
* JSON validity
* Maximum prompt size
* Forbidden content
* Tool-call structure

---

## Module 3: RAG pipeline

Learn:

* Document ingestion
* Document parsing
* Cleaning
* Chunking
* Metadata extraction
* Embedding generation
* Vector-index creation
* Index versioning
* Retrieval tests
* Re-indexing
* Incremental updates
* Access control
* Data deletion
* Source citation
* Freshness validation

Pipeline:

```text
Document change
→ Parse
→ Chunk
→ Embed
→ Build index
→ Retrieval evaluation
→ Publish index version
```

---

## Module 4: LLM evaluation

Build evaluation gates for:

* Answer correctness
* Relevance
* Groundedness
* Faithfulness
* Hallucination
* Source attribution
* Context recall
* Context precision
* Structured-output validity
* Tool-call correctness
* Toxicity
* PII leakage
* Prompt injection resistance
* Jailbreak resistance
* Refusal correctness
* Latency
* Token usage
* Cost per request

Evaluation types:

* Deterministic tests
* Rule-based validation
* Human evaluation
* LLM-as-judge
* Pairwise comparison
* Golden-dataset regression
* Production feedback

Do not make LLM-as-judge the only acceptance mechanism.

---

## Module 5: LLM deployment

Learn:

* Hosted model APIs
* Self-hosted open-source models
* CPU inference
* GPU inference
* Quantization
* Model loading
* GPU memory estimation
* Batching
* Streaming responses
* KV cache
* Context-window limits
* Request queues
* Concurrency limits
* Timeouts
* Retries
* Fallback models
* Rate limiting
* Tenant isolation

### EC2 deployment considerations

* GPU instance availability
* Model download time
* Model warm-up
* EBS size
* Container image size
* Health-check timing
* Graceful shutdown
* Request draining
* Autoscaling lag
* Model-cache persistence
* Capacity reservations

---

## Module 6: LLM observability

Capture:

* Request ID
* User or tenant ID, subject to privacy requirements
* Model
* Prompt version
* Retrieval version
* Number of retrieved chunks
* Input tokens
* Output tokens
* Latency
* Tool calls
* Safety decisions
* Errors
* Estimated cost
* User feedback

Never log raw sensitive prompts or personal data without a defined privacy and retention policy.

---

# 7. DevSecOps Topics to Add to Jenkins

A production pipeline should eventually contain:

```text
Source scan
→ Dependency scan
→ Secret scan
→ Unit tests
→ IaC scan
→ Container scan
→ SBOM
→ Image signing
→ Deployment
→ Runtime verification
```

Important topics:

* SAST
* Dependency vulnerability scanning
* Secret scanning
* Container scanning
* Infrastructure-as-Code scanning
* Software Bill of Materials
* Artifact signing
* Image provenance
* License compliance
* Supply-chain security
* Deployment attestations
* Approval records

Common tool categories:

* SonarQube
* Trivy
* Grype
* Syft
* Checkov
* tfsec
* Semgrep
* Gitleaks
* Cosign

---

# 8. Deployment Patterns You Should Practise

## Pattern 1: Direct deployment to one EC2 instance

```text
Jenkins
→ Copy application
→ Install dependencies
→ Restart systemd service
→ Test health endpoint
```

Use only for learning or simple non-critical applications.

---

## Pattern 2: Docker deployment to EC2

```text
Jenkins
→ Build image
→ Push to ECR
→ EC2 pulls image
→ Replace container
→ Health check
```

Good intermediate project.

---

## Pattern 3: Immutable EC2 deployment

```text
Jenkins
→ Build application
→ Build AMI
→ Create launch-template version
→ Refresh Auto Scaling Group
→ Validate new instances
```

Strong production pattern.

---

## Pattern 4: Blue-green deployment

```text
Blue environment: current model
Green environment: new model

Deploy green
→ Test green
→ Shift traffic
→ Monitor
→ Retire blue
```

---

## Pattern 5: Canary model deployment

```text
95% traffic → Existing model
5% traffic  → New model
```

Observe:

* Error rate
* Latency
* Prediction distribution
* Business results
* LLM quality
* Token cost

Then increase or roll back.

---

# 9. Essential Hands-On Projects

## Project 1: Basic EC2 application

Build:

* Ubuntu EC2
* FastAPI application
* `systemd` service
* Security group
* Health endpoint
* CloudWatch logs

---

## Project 2: Jenkins controller on EC2

Build:

* EC2 instance
* Jenkins installation
* Persistent EBS volume
* Nginx reverse proxy
* HTTPS
* Administrative user
* Backup procedure

---

## Project 3: Python CI pipeline

Pipeline:

```text
Git push
→ Checkout
→ Create virtual environment
→ Install requirements
→ Lint
→ Test
→ Coverage
→ Archive report
```

---

## Project 4: Docker and ECR pipeline

Pipeline:

```text
Git push
→ Test
→ Build Docker image
→ Scan
→ Push to ECR
```

Use the EC2 instance role rather than static AWS keys.

---

## Project 5: Automated EC2 deployment

Pipeline:

```text
Push image to ECR
→ Connect to target EC2
→ Pull image
→ Start container
→ Test /health
→ Roll back on failure
```

---

## Project 6: Dynamic Jenkins agents

Create:

* Small controller instance
* On-demand CPU agents
* Spot build agents
* GPU training agent
* Agent labels
* Automatic cleanup

---

## Project 7: MLOps training pipeline

Pipeline:

```text
Validate dataset
→ Train model
→ Evaluate
→ Compare with baseline
→ Register model
→ Human approval
→ Deploy
```

Record:

* Dataset version
* Git commit
* Hyperparameters
* Metrics
* Model artifact path

---

## Project 8: GPU training pipeline

Use:

* GPU EC2 instance
* Deep Learning AMI
* PyTorch
* S3 dataset
* Checkpointing
* CloudWatch GPU monitoring
* Automatic instance termination

---

## Project 9: RAG deployment pipeline

Pipeline:

```text
Upload documents
→ Parse
→ Chunk
→ Embed
→ Build vector index
→ Run retrieval tests
→ Deploy API
```

---

## Project 10: LLM evaluation pipeline

Create a golden dataset and evaluate:

* Correctness
* Groundedness
* JSON validity
* Hallucination
* Prompt injection
* Latency
* Token cost

Block production promotion when regression thresholds fail.

---

## Project 11: Auto Scaling inference service

Build:

* Launch template
* Auto Scaling Group
* Application Load Balancer
* Health checks
* CloudWatch alarms
* Rolling deployment
* Failure rollback

---

## Project 12: Production capstone

Build a complete platform:

```text
Git repository
→ Multibranch Jenkins Pipeline
→ CPU tests
→ Data validation
→ Model or LLM evaluation
→ Docker build
→ Security scan
→ ECR
→ Staging deployment
→ Smoke test
→ Manual approval
→ Production deployment
→ CloudWatch monitoring
→ Automated rollback
```

Required security:

* Private Jenkins controller
* IAM roles
* No static AWS keys
* IMDSv2
* Encrypted EBS
* HTTPS
* Restricted security groups
* Secrets Manager or Parameter Store
* Audit trail

Required reliability:

* Jenkins backup
* Build retention
* Health checks
* Deployment rollback
* Versioned artifacts
* Multi-AZ inference deployment
* Monitoring alarms

---

# 10. Recommended 16-Week Study Plan

| Week | Focus                             | Deliverable                           |
| ---- | --------------------------------- | ------------------------------------- |
| 1    | Linux, networking and Git         | Run and troubleshoot a Python service |
| 2    | EC2 fundamentals                  | Deploy FastAPI manually               |
| 3    | VPC, IAM and security groups      | Private and public subnet design      |
| 4    | EBS, AMIs and user data           | Reproducible EC2 launch               |
| 5    | Jenkins installation              | Jenkins controller on EC2             |
| 6    | Jenkinsfile and Pipeline          | Python CI pipeline                    |
| 7    | Webhooks and Multibranch Pipeline | Pull-request validation               |
| 8    | Docker and ECR                    | Build and publish image               |
| 9    | Jenkins agents                    | CPU and Docker agents                 |
| 10   | EC2 deployment                    | Automated application deployment      |
| 11   | ALB and Auto Scaling              | Scalable inference API                |
| 12   | MLOps training pipeline           | Train, evaluate and register model    |
| 13   | GPU and Spot workloads            | GPU training with checkpointing       |
| 14   | LLMOps and RAG                    | RAG build and evaluation pipeline     |
| 15   | Security, JCasC and backup        | Reproducible secured Jenkins          |
| 16   | Capstone                          | End-to-end production pipeline        |

---

# 11. Topic Priority by Role

## AI Engineer

### Must know

* Linux
* Git
* Python
* Docker
* EC2 basics
* Security groups
* IAM roles
* Jenkinsfile
* Testing
* ECR
* API deployment
* Logs and health checks
* LLM API integration
* Prompt and RAG testing

### Should know

* Auto Scaling
* Load balancers
* Secrets Manager
* GPU basics
* Canary deployment
* CloudWatch
* Infrastructure as Code

---

## MLOps Engineer

### Must know

Everything in the AI Engineer list, plus:

* Dataset versioning
* Data validation
* Training pipelines
* Experiment tracking
* Model registry
* Model evaluation
* GPU agents
* Spot handling
* Checkpointing
* Model deployment
* Drift monitoring
* Rollback
* Reproducibility
* Artifact lineage

### Advanced

* Distributed training
* EFA
* Cluster placement groups
* FSx for Lustre
* Multi-GPU environments
* Capacity planning

---

## LLMOps Engineer

### Must know

Everything in the AI Engineer list, plus:

* Prompt versioning
* RAG pipelines
* Embedding pipelines
* Vector-index versioning
* LLM evaluation
* Golden datasets
* Safety testing
* Prompt-injection testing
* Hallucination measurement
* Model routing
* Token-cost monitoring
* Latency monitoring
* Guardrail deployment
* Provider and model version tracking

### Advanced

* Self-hosted LLM deployment
* GPU inference
* Quantization
* Batching
* KV cache
* Model parallelism
* Autoscaling GPU inference
* Multi-model serving

---

# 12. Interview-Ready Areas

You should be able to explain clearly:

1. Why Jenkins needs controller and agent separation.
2. Why the Jenkins controller should not perform GPU training.
3. How a Git push triggers Jenkins.
4. How Jenkins authenticates to AWS without static keys.
5. How an EC2 instance role works.
6. How security groups differ from network ACLs.
7. How EBS differs from instance store.
8. How to preserve `JENKINS_HOME`.
9. How to recover Jenkins after instance failure.
10. How a Jenkinsfile is different from configuring a job manually.
11. How Multibranch Pipeline works.
12. How to protect credentials from pull-request builds.
13. How to build and push an image to ECR.
14. How to deploy the image to EC2.
15. How to implement rollback.
16. How Spot interruption affects training.
17. How to checkpoint and resume training.
18. How to evaluate a model before deployment.
19. How to evaluate an LLM prompt before deployment.
20. How to detect model or LLM regression.
21. How to scale an inference API.
22. How GPU autoscaling differs from CPU autoscaling.
23. How to secure Jenkins in a private subnet.
24. How to monitor Jenkins, EC2 and the deployed model.
25. How to trace a production response back to its code, model, prompt and dataset versions.

## Final learning order

Follow this sequence:

```text
Linux
→ Networking
→ Git
→ Docker
→ EC2
→ IAM
→ Jenkins installation
→ Jenkinsfile
→ Jenkins agents
→ ECR
→ EC2 deployment
→ Auto Scaling and ALB
→ Infrastructure as Code
→ MLOps pipelines
→ LLMOps pipelines
→ Security and observability
→ Distributed GPU topics
```

For job preparation, the most valuable final proof is one working repository containing a Jenkinsfile, Dockerfile, infrastructure code, automated tests, model or LLM evaluation, ECR publishing, EC2 deployment, monitoring and rollback.

[1]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html?utm_source=chatgpt.com "What is Amazon EC2? - Amazon Elastic Compute Cloud"
[2]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html?utm_source=chatgpt.com "Amazon EC2 instance types - Amazon Elastic Compute Cloud"
[3]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html?utm_source=chatgpt.com "Use instance metadata to manage your EC2 instance - Amazon Elastic Compute Cloud"
[4]: https://docs.aws.amazon.com/dlami/latest/devguide/what-is-dlami.html?utm_source=chatgpt.com "What is AWS Deep Learning AMIs? - AWS Deep Learning AMIs"
[5]: https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html?utm_source=chatgpt.com "What is Amazon Elastic Block Store? - Amazon EBS"
[6]: https://docs.aws.amazon.com/fsx/latest/LustreGuide/what-is.html?utm_source=chatgpt.com "What is Amazon FSx for Lustre? - FSx for Lustre"
[7]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-security-group.html?utm_source=chatgpt.com "Create a security group for your Amazon EC2 instance - Amazon Elastic Compute Cloud"
[8]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html?utm_source=chatgpt.com "IAM roles for Amazon EC2 - Amazon Elastic Compute Cloud"
[9]: https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-setting-up-ec2.html?utm_source=chatgpt.com "Managing EC2 instances with Systems Manager - AWS Systems Manager"
[10]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html?utm_source=chatgpt.com "Access instance metadata for an EC2 instance - Amazon Elastic Compute Cloud"
[11]: https://docs.aws.amazon.com/autoscaling/ec2/userguide/tutorial-ec2-auto-scaling-load-balancer.html?utm_source=chatgpt.com "Tutorial: Set up a scaled and load-balanced application - Amazon EC2 Auto Scaling"
[12]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html?utm_source=chatgpt.com "Spot Instances - Amazon Elastic Compute Cloud"
[13]: https://docs.aws.amazon.com/us_en/AWSEC2/latest/UserGuide/ec2-capacity-reservations.html?utm_source=chatgpt.com "Reserve compute capacity with EC2 On-Demand Capacity Reservations - Amazon Elastic Compute Cloud"
[14]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html?utm_source=chatgpt.com "Collect metrics, logs, and traces using the CloudWatch agent - Amazon CloudWatch"
[15]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html?utm_source=chatgpt.com "Placement groups for your Amazon EC2 instances - Amazon Elastic Compute Cloud"
[16]: https://www.jenkins.io/doc/book/pipeline/?utm_source=chatgpt.com "Pipeline"
[17]: https://www.jenkins.io/doc/pipeline/tour/agents/?utm_source=chatgpt.com "Defining execution environments"
[18]: https://www.jenkins.io/doc/book/security/credentials/?utm_source=chatgpt.com "Credentials"
[19]: https://www.jenkins.io/doc/book/security/securing-org-folders-and-multibranch-pipelines/?utm_source=chatgpt.com "Securing SCM credentials for Organization Folders and Multibranch Pipelines"
[20]: https://www.jenkins.io/doc/book/managing/plugins/?utm_source=chatgpt.com "Managing Plugins"
[21]: https://www.jenkins.io/doc/book/managing/casc/?utm_source=chatgpt.com "Configuration as Code"
[22]: https://www.jenkins.io/doc/book/system-administration/backing-up/?utm_source=chatgpt.com "Backing-up/Restoring Jenkins"
[23]: https://www.jenkins.io/doc/book/scaling/?utm_source=chatgpt.com "Scaling Jenkins"
[24]: https://www.jenkins.io/doc/book/system-administration/monitoring/?utm_source=chatgpt.com "Monitoring Jenkins"
[25]: https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html?utm_source=chatgpt.com "What is CodeDeploy? - AWS CodeDeploy"
