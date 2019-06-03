## Install Minikube and dependency required for it.


- Install required version of Docker for minikube. To run the script make sure are in this directory only.

```command
bash docker.sh
```

- Install kubectl, Minikube and other binaries on the Your infrastructure. This script will also bring up your minikube kubernetes cluster.

```command
bash minikube.sh
```

- Verify Kubernetes cluster is up and running.

```command
kubectl get node
```
```
NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   2m4s   v1.14.2
```

- Install Helm binaries.

```command
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh && chmod 700 get_helm.sh && bash get_helm.sh
```

- Create Service Account and RBAC rule.

```command
kubectl apply -f helm-rbac.yaml
```

-  Initialize the Helm.

```command
helm init --service-account tiller
```

- Install Ingress.

```command
minikube addons enable ingress
```
