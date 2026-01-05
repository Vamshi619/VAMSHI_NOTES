# Wealth AI Platform API - Task Based (S3 Storage)

A high-performance, scalable API for portfolio analysis using AI with S3 persistent storage.

## 🚀 Features

- **Async Task Processing** - Non-blocking background workers
- **S3 Persistent Storage** - Tasks survive pod restarts
- **Horizontal Scaling** - Multiple pods work independently (no race conditions)
- **Parallel Section Processing** - Multiple sections processed simultaneously
- **Auto-Recovery** - Pending tasks re-queued after restart
- **Auto-Cleanup** - S3 lifecycle policy deletes old tasks

## 📋 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         FastAPI App                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   Pod 1     │  │   Pod 2     │  │   Pod 3     │              │
│  │  (async)    │  │  (async)    │  │  (async)    │              │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
│         └────────────────┼────────────────┘                      │
│                          │                                       │
│                    Async S3 Client                               │
└──────────────────────────┼───────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                        S3 Bucket                                 │
│  tasks/                                                          │
│  ├── task_20251230_001.json                                     │
│  ├── task_20251230_002.json                                     │
│  └── ...                                                         │
└─────────────────────────────────────────────────────────────────┘
```

## 🛠️ Setup

### 1. Prerequisites

- Python 3.11+
- AWS Account with S3 access
- Your existing AI platform credentials

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure Environment

Copy the example environment file and fill in your values:

```bash
cp .env.example .env
```

Edit `.env` with your configuration:

```bash
# AWS Credentials (REQUIRED)
AWS_ACCESS_KEY_ID=AKIAXXXXXXXXXXXXXXXXXX
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_REGION=ap-south-1

# S3 Configuration
S3_BUCKET_NAME=wealth-ai-tasks
S3_PREFIX=tasks/

# Your existing API config
BASE_URL=https://your-ai-platform-url.com
API_KEY=your-api-key
TENANT_ID=idigi
```

### 4. Run Locally

```bash
python api_task_1.py
```

Or with uvicorn:

```bash
uvicorn api_task_1:app --host 0.0.0.0 --port 8000 --reload
```

### 5. Access API

- **API Documentation**: http://localhost:8000/docs
- **Health Check**: http://localhost:8000/ping
- **Dashboard**: http://localhost:8000/dashboard

## 📡 API Endpoints

### Submit Task
```bash
POST /task/submit
{
  "username": "user@example.com",
  "password": "xxx",
  "data": [
    {"Market Value": "12000000", "Top Asset": "Equity (65%)", ...},
    {"top_movers_data": "..."},
    {"total_portfolio_mv": 1000000, "assets": [...]}
  ]
}
```

### Check Status
```bash
GET /task/{task_id}/status
```

### Get Result
```bash
GET /task/{task_id}/result
```

### List Tasks
```bash
GET /tasks?limit=20&status=completed
```

## 🐳 Docker

### Build Image

```bash
docker build -t wealth-ai-api:latest .
```

### Run Container

```bash
docker run -d \
  -p 8000:8000 \
  -e AWS_ACCESS_KEY_ID=xxx \
  -e AWS_SECRET_ACCESS_KEY=xxx \
  -e AWS_REGION=ap-south-1 \
  -e S3_BUCKET_NAME=wealth-ai-tasks \
  -e BASE_URL=https://your-api.com \
  -e API_KEY=your-key \
  wealth-ai-api:latest
```

## ☸️ Kubernetes Deployment

### Deploy to Kubernetes

1. Update secrets in `k8s-deployment.yaml`
2. Apply the deployment:

```bash
kubectl apply -f k8s-deployment.yaml
```

### Verify Deployment

```bash
kubectl get pods -l app=wealth-ai-api
kubectl logs -f deployment/wealth-ai-api
```

## 📊 Monitoring

### Health Check Response

```json
{
  "status": "healthy",
  "workers": 10,
  "workers_started": true,
  "queue_size": 0,
  "total_tasks": 15,
  "storage": {
    "status": "healthy",
    "bucket": "wealth-ai-tasks",
    "cache_size": 15
  }
}
```

## 📁 File Structure

```
.
├── api_task_1.py                      # FastAPI application
├── task_manager.py                    # Task manager with workers
├── task_storage_s3_async_individual.py # S3 storage class
├── task_models.py                     # Pydantic models
├── pf_interface_1.py                  # External API interface
├── data_model_3.py                    # Data models
├── section_config.py                  # Section detection
├── config.json                        # Section configuration
├── requirements.txt                   # Python dependencies
├── Dockerfile                         # Container build
├── k8s-deployment.yaml               # Kubernetes deployment
├── .env.example                       # Environment template
└── README.md                          # This file
```

## 🔄 Migration from Local JSON

If migrating from local JSON storage:

1. The new system will automatically initialize an empty S3 bucket
2. Old tasks in `tasks.json` will NOT be migrated automatically
3. To migrate old tasks, you can create a migration script or start fresh

## ⚠️ Important Notes

1. **AWS Credentials**: Never commit real credentials to version control
2. **S3 Bucket**: The bucket will be created automatically if it doesn't exist
3. **Lifecycle Policy**: Tasks are automatically deleted after 30 days (configurable)
4. **Pod Restarts**: Pending tasks are automatically re-queued on startup
5. **Horizontal Scaling**: Multiple pods can run simultaneously without conflicts

## 🐛 Troubleshooting

### S3 Connection Issues

```bash
# Check AWS credentials
aws s3 ls s3://your-bucket-name

# Verify environment variables
echo $AWS_ACCESS_KEY_ID
echo $AWS_REGION
```

### Workers Not Starting

```bash
# Check logs
kubectl logs -f deployment/wealth-ai-api

# Check health endpoint
curl http://localhost:8000/ping
```

### Tasks Stuck in Pending

```bash
# Check queue size
curl http://localhost:8000/ping | jq '.queue_size'

# Check workers
curl http://localhost:8000/ping | jq '.workers_started'
```

## 📝 License

Proprietary - Intellect Design

## 👥 Authors

- Wealth AI Platform Team
