# Simple Notes App for TWS Community
This is a simple notes app built with React and Django.

## Requirements
1. Python 3.9
2. Node.js
3. React

## Installation
1. Clone the repository
```
git clone https://github.com/LondheShubham153/django-notes-app.git
```

2. Build the app
```
docker build -t notes-app .
```

3. Run the app
```
docker run -d -p 8000:8000 notes-app:latest
```

## Nginx

Install Nginx reverse proxy to make this application available

`sudo apt-get update`
`sudo apt install nginx`

## Kubernetes

The manifests in `k8s/` deploy the app and a MySQL database. Create the database credentials Secret once in the `notes-app` namespace (do not commit credentials), then apply the manifests:

```powershell
kubectl apply -f k8s/namespace.yml
kubectl create secret generic notes-app-db-credentials -n notes-app `
  --from-literal=DB_NAME=notesapp `
  --from-literal=DB_USER=notesapp `
  --from-literal=DB_PASSWORD=REPLACE_WITH_STRONG_PASSWORD `
  --from-literal=MYSQL_ROOT_PASSWORD=REPLACE_WITH_STRONG_ROOT_PASSWORD `
  --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f k8s/mysql.yml
kubectl apply -f k8s/deployment.yml
kubectl apply -f k8s/service.yml
kubectl port-forward -n notes-app svc/notes-app-service 8000:8000
```

Use strong passwords and keep them out of source control. The app runs Django migrations at startup; MySQL data is stored in the `notes-app-db-data` PersistentVolumeClaim.
