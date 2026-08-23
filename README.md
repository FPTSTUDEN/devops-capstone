Here's the minimally modified README.md with the deployment directories clearly separated:

```markdown
# DevOps Capstone — Account Microservice

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Python 3.9](https://img.shields.io/badge/Python-3.9-green.svg)](https://shields.io/)
![Build Status](https://github.com/<OWNER>/devops-capstone-project/actions/workflows/ci-build.yaml/badge.svg)

A fully functional **RESTful Account Microservice** built with Flask and PostgreSQL, containerized with Docker, and deployed to Kubernetes. This project is the capstone of the IBM DevOps and Software Engineering Professional Certificate.

---

## 📋 Table of Contents

- [What Was Implemented](#-what-was-implemented)
- [Project Structure](#-project-structure)
- [API Endpoints](#-api-endpoints)
- [Prerequisites](#-prerequisites)
- [Option 1: Run Locally (Python)](#-option-1-run-locally-python)
- [Option 2: Run with Kubernetes (Kind - Local)](#-option-2-run-with-kubernetes-kind---local)
- [Option 3: Deploy to OpenShift (IBM Cloud)](#-option-3-deploy-to-openshift-ibm-cloud)
- [Testing the API](#-testing-the-api)
- [Changes Made](#-changes-made)

---

## ✅ What Was Implemented

The original template only had a `POST /accounts` (Create) endpoint. The following were **added** to complete the full CRUD REST API:

| Feature | Status |
|---|---|
| `GET /accounts` — List all accounts | ✅ Implemented |
| `GET /accounts/<id>` — Read a single account | ✅ Implemented |
| `PUT /accounts/<id>` — Update an account | ✅ Implemented |
| `DELETE /accounts/<id>` — Delete an account | ✅ Implemented |
| `Dockerfile` — Containerize the application | ✅ Added |
| `deploy-local/postgresql.yaml` — PostgreSQL on Kind | ✅ Added |
| `deploy-local/deployment.yaml` — App deployment on Kind | ✅ Added |
| `deploy-local/service.yaml` — Expose app via NodePort | ✅ Added |
| `deploy/postgresql-ephemeral-template.json` — PostgreSQL on OpenShift | ✅ Added |
| `deploy/deployment.yaml` — App deployment on OpenShift | ✅ Added |
| `deploy/service.yaml` — Expose app via OpenShift Service | ✅ Added |
| `Makefile` fix — PostgreSQL 18+ Docker compatibility | ✅ Fixed |
| `requirements.txt` fix — psycopg2-binary for Python 3.9 | ✅ Fixed |

---

## 📁 Project Structure

```
devops-capstone/
├── service/                  ← The Flask microservice package
│   ├── common/               ← Shared log and error handlers
│   ├── config.py             ← Flask + SQLAlchemy configuration
│   ├── models.py             ← Account database model (SQLAlchemy)
│   └── routes.py             ← All REST API route handlers ← MAIN CHANGES HERE
│
├── deploy/                   ← OpenShift (IBM Cloud) manifests ← ORIGINAL
│   ├── postgresql-ephemeral-template.json
│   ├── deployment.yaml
│   └── service.yaml
│
├── deploy-local/             ← Kind (Local) manifests ← NEW
│   ├── postgresql.yaml       ← PostgreSQL Deployment + Service
│   ├── deployment.yaml       ← Account Service Deployment
│   └── service.yaml          ← NodePort Service to expose the app
│
├── tests/                    ← Unit and integration tests
│   ├── factories.py
│   ├── test_models.py
│   └── test_routes.py
│
├── Dockerfile                ← Docker image definition ← NEW
├── Makefile                  ← Dev commands (run, db, build, push)
├── Procfile                  ← gunicorn startup config
├── requirements.txt          ← Python dependencies
└── .flaskenv                 ← Flask environment variables
```

---

## 🔌 API Endpoints

Base URL (local): `http://localhost:5005` or `http://localhost:8085` (Kubernetes)

| Method | Endpoint | Description | Request Body |
|--------|----------|-------------|--------------|
| `GET` | `/` | Service info | — |
| `GET` | `/health` | Health check | — |
| `GET` | `/accounts` | List all accounts | — |
| `POST` | `/accounts` | Create a new account | JSON body (see below) |
| `GET` | `/accounts/<id>` | Get account by ID | — |
| `PUT` | `/accounts/<id>` | Update an account | JSON body (see below) |
| `DELETE` | `/accounts/<id>` | Delete an account | — |

### Account JSON Schema

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "address": "123 Main Street",
  "phone_number": "555-1234"
}
```

> `phone_number` is optional. All other fields are required.

---

## 🛠 Prerequisites

Make sure you have the following installed:

| Tool | Purpose | Install |
|------|---------|---------|
| Python 3.9 | Run the service locally | `brew install python@3.9` |
| Docker Desktop | Run PostgreSQL + build images | [docker.com](https://www.docker.com/products/docker-desktop) |
| kind | Local Kubernetes cluster | `brew install kind` |
| kubectl | Kubernetes CLI | `brew install kubectl` |
| oc | OpenShift CLI (for Option 3) | [OpenShift CLI](https://docs.openshift.com/container-platform/4.12/cli_reference/openshift_cli/getting-started-cli.html) |

---

## 🖥 Option 1: Run Locally (Python)

This runs the service directly on your Mac using a Python virtual environment.

### Step 1 — Set up Python 3.9 virtual environment

```bash
/opt/homebrew/opt/python@3.9/bin/python3.9 -m venv ~/venv39
source ~/venv39/bin/activate
pip install -r requirements.txt
```

### Step 2 — Start PostgreSQL with Docker

```bash
make db
```

> This starts a PostgreSQL container on port 5432.

### Step 3 — Run the service

```bash
PORT=5005 make run
```

### Step 4 — Access the service

Open your browser or use curl:
```
http://localhost:5005
```

Expected output:
```json
{"name": "Account REST API Service", "version": "1.0"}
```

---

## ☸️ Option 2: Run with Kubernetes (Kind - Local)

This runs the service inside a local Kubernetes cluster using **Kind**.

### Step 1 — Create the Kind cluster (if not already running)

```bash
make cluster
```

### Step 2 — Build the Docker image

```bash
make build
```

### Step 3 — Load the image into the cluster

```bash
kind load docker-image accounts:1.0 --name mlops-local
```

### Step 4 — Deploy to Kubernetes

```bash
kubectl apply -f deploy-local/
```

This creates:
- A `postgresql` Deployment + ClusterIP Service
- An `account-service` Deployment
- A `NodePort` Service for the account service

### Step 5 — Verify everything is running

```bash
kubectl get pods
```

Expected output:
```
NAME                             READY   STATUS    RESTARTS   AGE
account-service-xxx              1/1     Running   0          30s
postgresql-xxx                   1/1     Running   0          30s
```

### Step 6 — Forward the port to your machine

```bash
kubectl port-forward svc/account-service 8085:8080
```

### Step 7 — Access the service

```
http://localhost:8085
```

Expected output:
```json
{"name": "Account REST API Service", "version": "1.0"}
```

### To update and redeploy after code changes:

```bash
make build
kind load docker-image accounts:1.0 --name mlops-local
kubectl rollout restart deployment/account-service
kubectl rollout status deployment/account-service
```

---

## ☁️ Option 3: Deploy to OpenShift (IBM Cloud)

This deploys the service to OpenShift on IBM Cloud using the original lab instructions.

### Step 1 — Ensure you're logged into OpenShift

```bash
oc login <your-openshift-cluster-url>
```

### Step 2 — Deploy PostgreSQL using the ephemeral template

```bash
oc create -f deploy/postgresql-ephemeral-template.json
oc new-app postgresql-ephemeral
```

### Step 3 — Deploy the account service

```bash
oc create -f deploy/deployment.yaml
oc create -f deploy/service.yaml
```

### Step 4 — Expose the service via a route

```bash
oc expose service accounts --name=accounts --edge-termination
```

### Step 5 — Get the route URL

```bash
oc get routes
```

### Step 6 — Access the service

Open the route URL in your browser or use curl:
```bash
curl <route-url>
```

Expected output:
```json
{"name": "Account REST API Service", "version": "1.0"}
```

---

## 🧪 Testing the API

You can test all endpoints using `curl`:

### Create an account
```bash
curl -X POST http://localhost:8085/accounts \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com","address":"123 Main St","phone_number":"555-1234"}'
```

### List all accounts
```bash
curl http://localhost:8085/accounts
```

### Read a specific account
```bash
curl http://localhost:8085/accounts/1
```

### Update an account
```bash
curl -X PUT http://localhost:8085/accounts/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane Doe","email":"jane@example.com","address":"456 Oak Ave","phone_number":"555-9999"}'
```

### Delete an account
```bash
curl -X DELETE http://localhost:8085/accounts/1
# Returns: HTTP 204 No Content (success, no body)
```

### Health check
```bash
curl http://localhost:8085/health
# Returns: {"status": "OK"}
```

---

## 🔧 Changes Made

### 1. `service/routes.py` — Implemented 4 missing REST endpoints

The original template had placeholder comments. These were replaced with working implementations:

- **`GET /accounts`** — Queries all accounts from the database and returns them as a JSON list
- **`GET /accounts/<id>`** — Finds an account by ID, returns 404 if not found
- **`PUT /accounts/<id>`** — Updates an existing account, returns 404 if not found
- **`DELETE /accounts/<id>`** — Deletes an account if it exists, returns HTTP 204

### 2. `Dockerfile` — New file

Added a `Dockerfile` to containerize the application using `python:3.9-slim` as the base image and `gunicorn` as the WSGI server.

### 3. `deploy-local/postgresql.yaml` — New file

Kubernetes manifest that runs PostgreSQL inside the Kind cluster with:
- `PGDATA` environment variable set for Postgres 18+ compatibility
- ClusterIP service so the app pod can connect to it via `postgresql:5432`

### 4. `deploy-local/deployment.yaml` — New file

Kubernetes manifest for the Account Service on Kind:
- Uses the `accounts:1.0` Docker image
- Passes `DATABASE_URI` environment variable pointing to the in-cluster PostgreSQL

### 5. `deploy-local/service.yaml` — New file

Kubernetes `NodePort` Service that exposes the account service on port 8080 inside the Kind cluster.

### 6. `deploy/` — OpenShift manifests (original)

The `deploy/` directory contains the original OpenShift manifests from the lab:
- `postgresql-ephemeral-template.json` — OpenShift template for PostgreSQL
- `deployment.yaml` — Account Service deployment on OpenShift
- `service.yaml` — OpenShift Service exposing the app

### 7. `Makefile` — Fixed `db` target

Added `-e PGDATA=/var/lib/postgresql/data/pgdata` to the `docker run` command. This fixes a crash with the latest `postgres:alpine` image (version 18+) which requires data to be stored in a subdirectory.

### 8. `requirements.txt` — Fixed psycopg2 version

Changed `psycopg2-binary==2.9.3` to `psycopg2-binary>=2.9.9` to support Apple Silicon (M1/M2/M3) Macs and Python 3.9 on modern systems.

---

## 👤 Author

Implemented by **@sathvikvelapaka** as part of the IBM DevOps Capstone Project.

Original template by John Rofrano, IBM Research.

---

## 📄 License

Licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.
