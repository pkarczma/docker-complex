# Fibonacci Calculator - Containerized

A multi-tier web application that calculates Fibonacci numbers using a microservices architecture. Built with React, Express.js, Redis, and PostgreSQL, containerized with Docker and deployed on AWS Elastic Beanstalk.

## 🏗️ Architecture

This application demonstrates a modern containerized microservices architecture:

- **Frontend**: React.js single-page application
- **Backend API**: Express.js REST API server  
- **Worker Service**: Background Fibonacci calculator
- **Cache**: Redis for storing calculation results
- **Database**: PostgreSQL for persistent storage
- **Reverse Proxy**: Nginx for request routing
- **Containerization**: Docker & Docker Compose
- **Deployment**: AWS Elastic Beanstalk
- **CI/CD**: GitHub Actions

## 🚀 Features

- Calculate Fibonacci numbers for any index (up to 40)
- Real-time display of previously calculated values
- Responsive React frontend with routing
- Background processing for intensive calculations
- Persistent storage of calculation history
- Scalable microservices architecture

## 📋 Tech Stack

| Component | Technology |
|-----------|------------|
| **Frontend** | React 18, React Router, Axios |
| **Backend** | Node.js, Express.js, CORS |
| **Worker** | Node.js background service |
| **Database** | PostgreSQL |
| **Cache** | Redis |
| **Reverse Proxy** | Nginx |
| **Containerization** | Docker, Docker Compose |
| **Cloud Platform** | AWS Elastic Beanstalk |
| **CI/CD** | GitHub Actions |

## 🛠️ Development Setup

### Prerequisites

- Docker and Docker Compose installed
- Git

### Quick Start

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd fibonacci-calculator-containerized
   ```

2. **Start the development environment**
   ```bash
   docker compose -f docker-compose-dev.yml up --build
   ```

3. **Access the application**
   - Open your browser to: `http://localhost:3050`
   - The application includes:
     - React development server with hot reloading
     - Express API server with nodemon
     - PostgreSQL database
     - Redis cache
     - Nginx reverse proxy

### Development Workflow

The development setup includes:
- **Hot reloading** for both React frontend and Express backend
- **Volume mounts** for real-time code changes
- **Separate services** for each component
- **Local database and cache** for development

### Stopping the Environment

```bash
docker compose -f docker-compose-dev.yml down
```

## 🔧 Application Flow

1. **User Input**: User enters a number in the React frontend
2. **API Request**: Frontend sends POST request to `/api/values`
3. **Data Storage**: 
   - Number stored in PostgreSQL database
   - Placeholder stored in Redis cache
4. **Background Processing**: Worker service calculates Fibonacci number
5. **Result Storage**: Calculated value stored in Redis
6. **Display**: Frontend fetches and displays results from both data stores

## 🚀 CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/deploy.yml`) automates:

### Build Phase
1. **Test Execution**: Runs React test suite
2. **Image Building**: Creates Docker images for all services:
   - `client` (React production build)
   - `server` (Express API)
   - `worker` (Fibonacci calculator)
   - `nginx` (Reverse proxy)

### Deployment Phase
3. **Image Registry**: Pushes images to Docker Hub
4. **Package Creation**: Creates deployment package with `docker-compose.yml`
5. **AWS Deployment**: Deploys to Elastic Beanstalk environment

### Triggered By
- Push to `main` branch
- Automatic deployment on successful builds

## 🚀 How AWS Deploys the Application

AWS Elastic Beanstalk uses the **Multi-Container Docker platform** to deploy your application:

1. **Deployment Package**: GitHub Actions creates `deploy.zip` containing `docker-compose.yml` and source code
2. **Container Orchestration**: EB reads `docker-compose.yml` and pulls pre-built images from Docker Hub:
   - `nginx` (port 80) - Routes traffic to internal services
   - `client` - React frontend served internally  
   - `server` - Express API connected to RDS/ElastiCache
   - `worker` - Background Fibonacci calculator
3. **Service Integration**: EB injects environment variables to connect containers to AWS managed services (RDS PostgreSQL, ElastiCache Redis)

The `docker-compose.yml` serves as the deployment blueprint, while the actual application code runs from the Docker images built during CI/CD.

## ☁️ AWS Infrastructure Setup

### Required AWS Services

1. **Elastic Beanstalk Application**
2. **RDS PostgreSQL Database**  
3. **ElastiCache Redis Cluster**
4. **VPC Security Groups**
5. **IAM User for Deployment**

### Detailed Setup Instructions

#### 1. Elastic Beanstalk Application

1. Navigate to AWS Elastic Beanstalk console
2. Click "Create Application"
3. Set Application Name: `multi-docker`
4. Platform: **Docker**
5. Environment: **Single Instance (free tier)**
6. Service Role: `aws-elasticbeanstalk-service-role`
7. Instance Profile: `aws-elasticbeanstalk-ec2-role`

#### 2. RDS PostgreSQL Database

1. Go to RDS console → "Create database"
2. Engine: **PostgreSQL**
3. Template: **Free tier**
4. Settings:
   - DB Instance identifier: `multi-docker-postgres`
   - Master Username: `postgres`
   - Master Password: `postgrespassword`
   - Initial database name: `fibvalues`
5. VPC: **Default VPC**

#### 3. ElastiCache Redis Cluster

1. Navigate to ElastiCache console
2. Create Redis OSS cache
3. Configuration:
   - Name: `multi-docker-redis`
   - Node type: `cache.t3.micro`
   - Replicas: `0`
   - Cluster Mode: **DISABLED**
4. **Important**: After creation, modify cache:
   - Transit encryption: Change from "Required" to "Preferred"

#### 4. Security Groups Configuration

1. **Create Security Group**:
   - Name: `multi-docker`
   - VPC: Default VPC
   - Inbound rule: Port range `5432-6379`, Source: `multi-docker` security group

2. **Apply to Services**:
   - **ElastiCache**: Modify cluster → Add `multi-docker` security group
   - **RDS**: Modify database → Add `multi-docker` security group  
   - **Elastic Beanstalk**: Configuration → Instances → Add `multi-docker` security group

#### 5. Environment Variables

In Elastic Beanstalk environment configuration, add:

| Variable | Value | Description |
|----------|-------|-------------|
| `REDIS_HOST` | `<elasticache-endpoint>` | Redis primary endpoint (without :6379) |
| `REDIS_PORT` | `6379` | Redis port |
| `PGUSER` | `postgres` | Database username |
| `PGPASSWORD` | `postgrespassword` | Database password |
| `PGHOST` | `<rds-endpoint>` | RDS database endpoint |
| `PGDATABASE` | `fibvalues` | Database name |
| `PGPORT` | `5432` | PostgreSQL port |

#### 6. IAM User for GitHub Actions

1. Create IAM user in AWS, e.g.: `docker-multi-github-actions`
2. Attach policy: `AdministratorAccess-AWSElasticBeanstalk`
3. Create access keys
4. Add to GitHub repository secrets:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_KEY`

### GitHub Repository Secrets

Required secrets for CI/CD pipeline:

```
AWS_ACCESS_KEY_ID=<your-access-key>
AWS_SECRET_KEY=<your-secret-key>
DOCKER_USERNAME=<your-dockerhub-username>
DOCKER_PASSWORD=<your-dockerhub-password>
```

## 📊 Monitoring and Troubleshooting

### Health Checks
- **Elastic Beanstalk**: Monitor environment health in AWS console
- **Application Logs**: View logs in EB environment dashboard
- **Container Logs**: Use `docker compose logs <service-name>` locally

### Common Issues
1. **504 Gateway Timeout**: Check security group configurations
2. **Database Connection**: Verify environment variables and RDS accessibility
3. **Redis Connection**: Ensure ElastiCache security groups and encryption settings

## 🔄 Deployment Process

1. **Make Changes**: Modify code in any service
2. **Commit & Push**: 
   ```bash
   git add .
   git commit -m "your changes"
   git push origin main
   ```
3. **Monitor**: Check GitHub Actions for build status
4. **Verify**: Visit Elastic Beanstalk URL when deployment completes

## 📁 Project Structure

```
├── client/                 # React frontend
│   ├── src/
│   ├── public/
│   ├── Dockerfile
│   └── Dockerfile.dev
├── server/                 # Express API
│   ├── index.js
│   ├── keys.js
│   ├── Dockerfile
│   └── Dockerfile.dev
├── worker/                 # Fibonacci calculator
│   ├── index.js
│   ├── keys.js
│   ├── Dockerfile
│   └── Dockerfile.dev
├── nginx/                  # Reverse proxy
│   ├── default.conf
│   ├── Dockerfile
│   └── Dockerfile.dev
├── docker-compose.yml      # Production configuration
├── docker-compose-dev.yml  # Development configuration
└── .github/workflows/
    └── deploy.yml          # CI/CD pipeline
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally with Docker Compose
5. Submit a pull request

## 📄 License

This project is part of a learning exercise demonstrating containerized microservices architecture and cloud deployment patterns.
