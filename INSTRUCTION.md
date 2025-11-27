# How to deploy and test the Todo app with Deployment and HPA

## 1. Build and push Docker image (one time, if not built yet)

From the `src` folder you can build and push the image to Docker Hub:

```bash
cd src

# Build image for Django Todo application
docker build -t ikulyk404/todoapp:3.0.0 .

# Push image to Docker Hub
docker push ikulyk404/todoapp:3.0.0

# From repository root
cd ..

# Create namespace
kubectl apply -f _infrastructure/namespace.yml

# (Optional) Pods and services reused for testing
kubectl apply -f _infrastructure/busybox.yml
kubectl apply -f _infrastructure/clusterIp.yml
kubectl apply -f _infrastructure/nodeport.yml

# Apply Deployment and HPA
kubectl apply -f _infrastructure/deployment.yml
kubectl apply -f _infrastructure/hpa.yml

# Check pods in namespace
kubectl get pods -n todoapp

# Check Deployment and ReplicaSets
kubectl get deploy -n todoapp
kubectl describe deploy todoapp -n todoapp

# Check services (ClusterIP / NodePort)
kubectl get svc -n todoapp

# Check HPA status
kubectl get hpa -n todoapp
kubectl describe hpa todoapp-hpa -n todoapp

# Open a shell inside the busybox pod
kubectl exec -it busybox -n todoapp -- sh

# From busybox container call the health endpoint via ClusterIP service
# (run inside the busybox shell)
wget -qO- http://todoapp-clusterip/api/health
# or
curl -s http://todoapp-clusterip/api/health

# Forward local port 8080 to the ClusterIP service inside the cluster
kubectl -n todoapp port-forward svc/todoapp-clusterip 8080:80

# After that, open in browser or with curl:
http://localhost:8080/api/health
http://localhost:8080/

# Get node IP:
kubectl get nodes -o wide

# Cleanup
kubectl delete -f _infrastructure/hpa.yml
kubectl delete -f _infrastructure/deployment.yml
kubectl delete -f _infrastructure/nodeport.yml
kubectl delete -f _infrastructure/clusterIp.yml
kubectl delete -f _infrastructure/busybox.yml
kubectl delete -f _infrastructure/namespace.yml

