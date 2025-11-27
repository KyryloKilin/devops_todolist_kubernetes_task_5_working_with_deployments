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
kubectl apply -f .infrastructure/namespace.yml

# (Optional) Pods and services reused for testing
kubectl apply -f .infrastructure/busybox.yml
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/nodeport.yml

# Apply Deployment and HPA
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml

# Check pods in namespace
kubectl get pods -n mateapp

# Check Deployment and ReplicaSets
kubectl get deploy -n mateapp
kubectl describe deploy mateapp -n mateapp

# Check services (ClusterIP / NodePort)
kubectl get svc -n mateapp

# Check HPA status
kubectl get hpa -n mateapp
kubectl describe hpa mateapp-hpa -n mateapp

# Open a shell inside the busybox pod
kubectl exec -it busybox -n mateapp -- sh

# From busybox container call the health endpoint via ClusterIP service
# (run inside the busybox shell)
wget -qO- http://mateapp-clusterip/api/health
# or
curl -s http://mateapp-clusterip/api/health

# Forward local port 8080 to the ClusterIP service inside the cluster
kubectl -n mateapp port-forward svc/mateapp-clusterip 8080:80

# After that, open in browser or with curl:
http://localhost:8080/api/health
http://localhost:8080/

# Get node IP:
kubectl get nodes -o wide

# Explanation of configuration choices
# Resource requests and limits

In deployment.yml each pod has resource requests and limits (for example):

requests:

cpu: 100m

memory: 128Mi

limits:

cpu: 300m

memory: 256Mi

Why this configuration:

requests describe the minimum guaranteed resources.
100m CPU и 128Mi пам€ти достаточно дл€ лЄгкого Django-приложени€ в простом учебном кластере.

limits защищают кластер от перерасхода ресурсов одним подом.
300m CPU и 256Mi пам€ти дают приложению запас при нагрузке, но не позвол€ют ему УсъестьФ слишком много ресурсов.

“акой баланс даЄт стабильную работу приложени€ и корректную работу автоскейлера по CPU.

5.2. HPA configuration

¬ hpa.yml использована конфигураци€ (пример):

minReplicas: 2

maxReplicas: 5

targetCPUUtilizationPercentage: 70

ќбъ€снение:

minReplicas: 2 Ч минимум два пода всегда запущены.
Ёто повышает отказоустойчивость: если один под упадЄт, приложение останетс€ доступным.

maxReplicas: 5 Ч ограничивает рост числа подов и защищает кластер от чрезмерного автоскейлинга.

targetCPUUtilizationPercentage: 70 Ч HPA увеличивает количество подов, когда средн€€ загрузка CPU выше ~70%.
Ёто разумный порог: поды успевают обрабатывать нагрузку, но при пиках трафика HPA масштабирует их число.

5.3. Deployment strategy (maxSurge и maxUnavailable)

¬ deployment.yml используетс€ стратеги€ RollingUpdate, например:

type: RollingUpdate

maxSurge: 1

maxUnavailable: 0

ѕочему так:

RollingUpdate позвол€ет обновл€ть приложение без просто€, по одному новому поду за раз.

maxSurge: 1 Ч при обновлении Kubernetes может создать один дополнительный под поверх желаемого количества.
Ёто позвол€ет сначала подн€ть новый под, убедитьс€, что он здоров, и только потом выключать старые.

maxUnavailable: 0 Ч во врем€ депло€ ни один существующий под не должен быть недоступен.
Ёто обеспечивает максимально возможную доступность приложени€ во врем€ обновлений.

“акое сочетание параметров идеально подходит дл€ учебного продакшен-подобного сценари€: обновлени€ происход€т плавно и без даунтайма.



# Cleanup
kubectl delete -f .infrastructure/hpa.yml
kubectl delete -f .infrastructure/deployment.yml
kubectl delete -f .infrastructure/nodeport.yml
kubectl delete -f .infrastructure/clusterIp.yml
kubectl delete -f .infrastructure/busybox.yml
kubectl delete -f .infrastructure/namespace.yml

