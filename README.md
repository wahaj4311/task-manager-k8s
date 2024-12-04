# Task Manager with Kubernetes

A simple task manager application demonstrating React with runtime environment variables in Kubernetes.

## Project Structure

```
.
├── backend/                 # Backend Node.js application
├── frontend/               # Frontend React application
├── kubernetes/             # Kubernetes configuration files
└── docs/                   # Documentation
```

## Features

- React frontend with runtime environment configuration
- Node.js backend with RESTful API
- Kubernetes deployment configuration
- Environment variable injection at runtime

## Prerequisites

- Node.js 16+
- Docker
- Kubernetes cluster (Minikube or similar)
- kubectl CLI tool

## Quick Start

1. Start your Kubernetes cluster:
```bash
minikube start
```

2. Apply Kubernetes configurations:
```bash
kubectl apply -f backend-deployment.yml
kubectl apply -f backend-service.yml
kubectl apply -f frontend-deployment.yml
kubectl apply -f frontend-service.yml
kubectl apply -f frontend-configmap.yml
```

3. Get the service URLs:
```bash
minikube service frontend-service --url
minikube service backend-service --url
```

## Environment Variables

### Frontend
- `REACT_APP_API_URL`: Backend API URL
- `REACT_APP_ENV`: Environment name (dev/prod)

### Backend
- `PORT`: Server port (default: 5000)

## Documentation

See [Runtime Environment Variables in React with Kubernetes](docs/react-kubernetes-env-vars.md) for detailed documentation about the environment variable handling pattern used in this project.

## Development

1. Frontend development:
```bash
cd frontend
npm install
npm start
```

2. Backend development:
```bash
cd backend
npm install
npm start
```

## Building and Deploying

1. Build Docker images:
```bash
# Frontend
cd frontend
docker build -t your-registry/frontend:latest .

# Backend
cd backend
docker build -t your-registry/backend:latest .
```

2. Push to registry:
```bash
docker push your-registry/frontend:latest
docker push your-registry/backend:latest
```

3. Deploy to Kubernetes:
```bash
kubectl apply -f kubernetes/
```

## License

MIT 