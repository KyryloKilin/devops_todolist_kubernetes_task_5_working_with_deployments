# How to deploy and test the Todo app with Deployment and HPA

## 1. Build and push Docker image (one time, if not built yet)

From the `src` folder you can build and push the image to Docker Hub:

```bash
cd src
docker build -t ikulyk404/todoapp:3.0.0 .
docker push ikulyk404/todoapp:3.0.0

# 1) Namespace
kubectl apply -f _infrastructure/namespace.yml

# 2) Pods and services used for testing
kubectl apply -f _infrastructure/busybox.yml
kubectl apply -f _infrastructure/clusterIp.yml
kubectl apply -f _infrastructure/nodeport.yml

# 3) Deployment, use and HPA
kubectl apply -f _infrastructure/deployment.yml
kubectl apply -f _infrastructure/hpa.yml
kubectl get hpa -n todoapp
kubectl describe hpa todoapp-hpa -n todoapp

# Check that everything is created
kubectl get pods -n todoapp
kubectl get deploy -n todoapp
kubectl get svc -n todoapp
kubectl get hpa -n todoapp

# busybox
kubectl exec -it busybox -n todoapp -- sh
while true; do curl -s http://todoapp-clusterip/api/health > /dev/null; done

# cleanup
kubectl delete -f _infrastructure/hpa.yml
kubectl delete -f _infrastructure/deployment.yml
kubectl delete -f _infrastructure/nodeport.yml
kubectl delete -f _infrastructure/clusterIp.yml
kubectl delete -f _infrastructure/busybox.yml
kubectl delete -f _infrastructure/namespace.yml
