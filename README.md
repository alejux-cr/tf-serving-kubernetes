# TensorFlow serving with Kubernetes in Local Env
Deployment for containers that run a model server, with scaling up/down based on CPU Utilization

## System requirements
- Docker
- curl
- Virtuabox (optional as docker can be specified as vm driver) https://www.virtualbox.org/wiki/Downloads
- kubectl https://kubernetes.io/docs/tasks/tools/
- Minikube https://minikube.sigs.k8s.io/docs/start/

## Setup
Setup done for Ubuntu Focal Fossa 20.04

First mount the location of the model inside Minikube's VM
```bash
cp -R ./saved_model_half_plus_two_cpu /var/tmp
```

Run Minikube: Init the VM with Virtualbox
```bash
minikube start --mount=True --mount-string="/var/tmp:/var/tmp" --vm-driver=virtualbox
```

MODEL_NAME and MODEL_PATH in the configmap.yaml are needed due to tensorflow/serving image is [configured](https://hub.docker.com/layers/tensorflow/serving/2.6.0/images/sha256-7e831f11c9ef928c09b4064a059066484079ac819991f162a938de0ad4b0fbd5?context=explore)

Create the object:
```bash
kubectl apply -f yaml/configmap.yaml
```
Get and validate the object:
```bash
kubectl describe cm tfserving-configs
```

Create a deployment:
```bash
kubectl apply -f yaml/deployment.yaml
```
Get deployment:
```bash
kubectl get deploy
```

Expose the deployment through a service:
Apply the service.yaml
```bash
kubectl apply -f yaml/service.yaml
```
Get the service:
```bash
kubectl get svc tf-serving-service
```
Access the deployment as sanity check with curl to send a row of inference requests to the Nodeport service:
```bash
curl -d '{"instances": [1.0, 2.0, 5.0]}' -X POST $(minikube ip):30001/v1/models/half_plus_two:predict
```
If the above doesn't work, try:
```bash
minikube ip
```
to get the IP address of the Minikube node

### Horizontal Pod Autoscaler
Enable Metrics Server in Minikube
```bash
minikube addons enable metrics-server
```
Run and wait for the deployment to be ready:
```bash
kubectl get deployment metrics-server -n kube-system
```
Then create the autoscaler by applying the autoscale.yaml
```bash
kubectl apply -f yaml/autoscale.yaml
```
Query the metrics
```bash
kubectl get hpa
```

### Stress tests
Run the bash script that sends requests to the app
```bash
/bin/bash request.sh
```

For monitoring, Minikube's built-in dashboard is useful:
```bash
minikube dashboard
```

### Cleanup
Destroy the resources:
```bash
kubectl delete -f yaml
```

Recreate all in one command:
```bash
kubectl apply -f yaml
```

If you want to destroy the VM run
```bash
minikube delete
```
