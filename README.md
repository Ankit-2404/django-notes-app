# Django Notes App

A Notes application built with **Django**, containerized with **Docker**, and deployed locally on **Kubernetes**. The project uses environment variables for database configuration.

> **Project note:** This README is based on the project details available so far. Review the setup commands, image name, Kubernetes resource names, and feature list against your actual source code before publishing.

## Contents

- [Overview](#overview)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Run Locally](#run-locally)
- [Run with Docker](#run-with-docker)
- [Deploy to Kubernetes](#deploy-to-kubernetes)
- [Useful Commands](#useful-commands)
- [Configuration and Security](#configuration-and-security)
- [Troubleshooting](#troubleshooting)
- [Future Improvements](#future-improvements)
- [License](#license)

## Overview

This repository demonstrates a containerized Django application and a Kubernetes deployment workflow. It is intended as a learning project for Python web development, containerization, and orchestration.

Update this section with the exact features implemented in your application, for example:
- Create, view, edit, or delete notes (include only if implemented).
- Persist application data in a database.
- Configure the application using environment variables.
- Build and run the application as a Docker image.
- Deploy the application and expose it through a Kubernetes Service.

## Technology Stack

- **Python / Django** — web application
- **Docker** — container image and runtime
- **Kubernetes** — deployment and service orchestration
- **MySQL** — database, if configured in your current setup
- **kubectl** — Kubernetes command-line tool

## Architecture

```text
Browser
   |
   v
Kubernetes Service
   |
   v
Django Pod(s)  ----->  MySQL Service / Container
   |
   v
Django application
```

The diagram is conceptual. Adjust it if your actual deployment uses different resources or a different database arrangement.

## Prerequisites

Install the tools used by your workflow:

- Python and pip
- Git
- Docker Desktop (or Docker Engine)
- kubectl
- A Kubernetes cluster, such as a local `kind` cluster
- MySQL, if your Django settings use MySQL

Check your installations:

```bash
python --version
pip --version
git --version
docker --version
kubectl version --client
```

## Run Locally

1. Clone your own GitHub repository:

   ```bash
   git clone https://github.com/Ankit-2404/django-notes-app.git
   cd <YOUR-REPOSITORY>
   ```

2. Create and activate a virtual environment.

   **Windows PowerShell:**
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   **macOS/Linux:**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. Install the dependencies. Use the dependency file included in the project (for example, `requirements.txt`):

   ```bash
   pip install -r requirements.txt
   ```

4. Create a local `.env` file based on `.env.example` and fill in the correct values. Never commit the real `.env` file.

5. Apply migrations:

   ```bash
   python manage.py migrate
   ```

6. Start Django:

   ```bash
   python manage.py runserver
   ```

7. Open `http://127.0.0.1:8000/`.

> These commands assume the project has `manage.py` at the repository root and the required database is available. If your project has a different layout or startup procedure, update these steps.

## Run with Docker

Build the image from the directory containing your Dockerfile:

```bash
docker build -t notes-app-k8s:latest .
```

Run it using the environment and networking configuration required by your application. For example, if your container listens on port 8000:

```bash
docker run --rm -p 8000:8000 --env-file .env notes-app-k8s:latest
```

The database must be reachable from the container. A database host such as `db_cont` only works if a container or service with that name is reachable on the same Docker network. Configure the hostname to match your actual Docker or Compose setup.

Open `http://localhost:8000/`.

## Deploy to Kubernetes

These are example commands; use the actual manifest paths and resource names in your repository.

1. Confirm your cluster is available:

   ```bash
   kubectl cluster-info
   kubectl get nodes
   ```

2. Build the image:

   ```bash
   docker build -t notes-app-k8s:latest .
   ```

3. If you use a local `kind` cluster, load the image into it (replace the cluster name if necessary):

   ```bash
   kind load docker-image notes-app-k8s:latest --name <YOUR-KIND-CLUSTER>
   ```

   If you publish the image to a registry instead, tag and push it, then reference the published image in the Deployment manifest:

   ```bash
   docker image tag notes-app-k8s:latest <DOCKERHUB-USERNAME>/notes-app-k8s:latest
   docker push <DOCKERHUB-USERNAME>/notes-app-k8s:latest
   ```

4. Apply your Kubernetes manifests. Update the paths to match your repository:

   ```bash
   kubectl apply -f k8s/
   ```

5. Inspect the resources:

   ```bash
   kubectl get deployments,pods,services -A
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```

6. Forward a local port to the Django Service. Replace `<NAMESPACE>` and `<SERVICE-NAME>` with the actual values:

   ```bash
   kubectl port-forward -n <NAMESPACE> service/<SERVICE-NAME> 8000:8000
   ```

   Keep the terminal running and open `http://localhost:8000/`. The Service's `port` and `targetPort` must match your manifest and Django container port. If the Service exposes port 80 and forwards to container port 8000, use `8000:80` for port-forwarding.

### Image availability

A Kubernetes cluster must be able to access the image. For a local `kind` cluster, either load the image into the cluster or use an accessible registry image. For private registries, configure image-pull credentials.

## Useful Commands

```bash
# Docker
docker images
docker ps
docker logs <CONTAINER-NAME>
docker build -t notes-app-k8s:latest .

# Kubernetes
kubectl get pods -A
kubectl get deployments -A
kubectl get services -A
kubectl describe pod <POD-NAME> -n <NAMESPACE>
kubectl logs <POD-NAME> -n <NAMESPACE>
kubectl get events -n <NAMESPACE> --sort-by=.metadata.creationTimestamp
```

## Configuration and Security

- Keep `.env` out of Git. Commit only `.env.example` with placeholder values.
- Do not publish passwords, API keys, database credentials, Django `SECRET_KEY` values, or Kubernetes Secrets.
- Use a strong, unique database password outside local learning environments.
- Set `DEBUG=False` and configure `ALLOWED_HOSTS` appropriately for a production deployment.
- Use Kubernetes Secrets or an appropriate secret manager for sensitive deployment configuration.
- Do not expose a database port publicly unless you have a specific, secured reason to do so.

## Troubleshooting

### Port-forward reports `connection refused`

The target port may not have an application listening on it. Check the Pod status and logs:

```bash
kubectl get pods -n <NAMESPACE>
kubectl logs <POD-NAME> -n <NAMESPACE>
kubectl describe pod <POD-NAME> -n <NAMESPACE>
```

Confirm Django starts with a bind address accessible from the Pod network, for example:

```bash
python manage.py runserver 0.0.0.0:8000
```

For production, prefer a production-grade WSGI/ASGI server rather than Django's development server.

### Pod cannot pull the image

Check the image name, tag, registry access, and `imagePullPolicy`. For `kind`, verify the image has been loaded into the correct cluster or published to a reachable registry.

### Django cannot connect to MySQL

Verify the database host, port, credentials, database name, network, and whether the database is ready before Django starts. Inside Kubernetes, use the database Service DNS name rather than assuming a Docker container name will resolve.

### Service is not reachable

```bash
kubectl get svc -n <NAMESPACE>
kubectl get endpoints -n <NAMESPACE>
kubectl get pods --show-labels -n <NAMESPACE>
```

Ensure the Service selector matches the Pod labels and the `targetPort` matches the port used by Django.

## Future Improvements

- Add automated tests and CI with GitHub Actions.
- Add health/readiness probes.
- Use a production WSGI/ASGI server.
- Add resource requests and limits.
- Store credentials using Kubernetes Secrets.
- Add screenshots or a short demo GIF of the actual application.

## License

No license has been selected for this repository yet. Add a `LICENSE` file only after choosing a license. Without a license, others generally do not receive explicit permission to reuse, modify, or distribute your code.
