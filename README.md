# my-argocd-app
my-argocd-app
# Prerequisites:
- Minikube installed and running on Ubuntu (WSL)
- ArgoCD installed and configured
- GitHub account and repository
- DockerHub account
- kubectl configured to access Minikube
# 1. Set Up a Simple Application
We'll create a simple Node.js application as an example.
1. Create a directory for your app and navigate into it:
```bash
mkdir my-argocd-app
cd my-argocd-app
```
2. Create a package.json file:
```bash
{
  "name": "my-argocd-app",
  "version": "1.0.0",
  "description": "A simple Node.js app",
  "main": "app.js",
  "dependencies": {
    "express": "^4.17.1"
  },
```
3.Create the app.js file:
```bash
const express = require('express');
const app = express();
const port = 8080;

app.get('/', (req, res) => {
  res.send('Hello from ArgoCD app!');
});

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```
4.Install the dependencies:
```bash
npm install
```
# 2. Create a Dockerfile
Next, create a Dockerfile to containerize your Node.js app.
```bash
# Use official Node.js image
FROM node:14

# Set working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy all files
COPY . .

# Expose the app port
EXPOSE 8080

# Command to run the app
CMD ["npm", "start"]
```
# 3. Build and Push Docker Image to DockerHub
1. Build the Docker image:
```bash
docker build -t <your-dockerhub-username>/my-argocd-app:v1 .
```
2. Log in to DockerHub:
```bash
docker login
```
3. Push the image to DockerHub:
```bash
docker push <your-dockerhub-username>/my-argocd-app:v1
```
# 4. Push the Code to GitHub
```bash
git init
git add .
git commit -m "Initial commit for ArgoCD app"
```
2. Create a GitHub repository (e.g., my-argocd-app).
3. Link your local repository to GitHub and push:
```bash
git remote add origin https://github.com/<your-github-username>/my-argocd-app.git
git push -u origin master
```
# 5. Create Kubernetes Deployment and Service for ArgoCD
1. Create a k8s directory:
```bash
mkdir k8s
cd k8s
```
2. Create a Kubernetes deployment.yaml file for your app:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-argocd-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-argocd-app
  template:
    metadata:
      labels:
        app: my-argocd-app
    spec:
      containers:
      - name: my-argocd-app
        image: <your-dockerhub-username>/my-argocd-app:v1
        ports:
        - containerPort: 8080
```
3. Create a Kubernetes service.yaml to expose the app:
```bash
apiVersion: v1
kind: Service
metadata:
  name: my-argocd-app
spec:
  selector:
    app: my-argocd-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```
4. Push the k8s/ directory to your GitHub repository.
# 6. Configure ArgoCD for Deployment
1. Create an ArgoCD Application:
  - Open the ArgoCD UI (you can access it via Minikube):
    ```bash
    minikube service argocd-server -n argocd
    
    ```
  - Log in using the default credentials (admin / get the password using kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | 
    base64 -d).
- Create a new Application in ArgoCD pointing to your GitHub repository:
  - Source: GitHub repository URL (e.g., https://github.com/<your-github-username>/my-argocd-app)
  - Path: k8s
  - Cluster: https://kubernetes.default.svc
  - Namespace: default
2. Apply the Application in ArgoCD:
   - Once the application is configured, ArgoCD will automatically detect the changes and deploy your app to the cluster.

# 7. Access the Application
 1. In Minikube, expose the app externally:
```bash
kubectl expose deployment my-argocd-app --type=LoadBalancer --name=my-argocd-app-service
kubectl expose deployment my-argocd-app --type=LoadBalancer --name=my-argocd-app-service -n app
minikube service my-argocd-app-service -n app --url
```
2. Get the URL to access the app:
3. Open the URL in your browser, and you should see the message "Hello from ArgoCD app!"
# 8. Automation with ArgoCD Sync
ArgoCD will automatically sync the app with the changes in your GitHub repository. Every time you push new changes (e.g., new Docker image version, code updates), ArgoCD will sync and redeploy the app.





