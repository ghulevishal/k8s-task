## Monitoring with Prometheus

#### Prerequisites : Helm must be installed and tiller pod must be running in your kubernetes cluster.


- Install the Prometheus with Helm chart.

```command
helm install --name prometheus --set server.service.type=NodePort stable/prometheus
```

- Get the list of the Pods.

```command
kubectl get pods
```
```output
NAME                                             READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-7c8d6b6754-mgtr2         2/2     Running   0          3m
prometheus-kube-state-metrics-74d5c694c7-j88h6   1/1     Running   0          3m
prometheus-node-exporter-rl5jj                   1/1     Running   0          3m
prometheus-pushgateway-d5fdc4f5b-7pzv6           1/1     Running   0          3m
prometheus-server-c946c7f8-b5fmt                 2/2     Running   0          3m
```

- Get the list of the services.

```command
kubectl get svc
```
```output
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes                      ClusterIP   10.96.0.1       <none>        443/TCP        1h
prometheus-alertmanager         ClusterIP   10.98.205.151   <none>        80/TCP         18s
prometheus-kube-state-metrics   ClusterIP   None            <none>        80/TCP         18s
prometheus-node-exporter        ClusterIP   None            <none>        9100/TCP       18s
prometheus-pushgateway          ClusterIP   10.110.53.233   <none>        9091/TCP       17s
prometheus-server               NodePort    10.96.194.14    <none>        80:30469/TCP   17s
```

Now try to access the Prometheus UI by using Pulic IP and NodePort. 


## Install Grafana Helm chart

```command
helm install --name grafana --set service.type=NodePort stable/grafana
```

- Get the list of the Pods.

```command
kubectl get pods | grep grafana
```
```
grafana-57bb579f79-dzz5w                         1/1     Running   0          27s
```

- List the grafan service

```command
kubectl get svc | grep grafana
```
```
grafana                         NodePort    10.97.144.71     <none>        80:31037/TCP   69s
```

- Get the password for Grafana dashboard

```command
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Using the `admin` as username and above derived password you can login to Grafana dashboard using nodeport and public ip.
## Clean UP.

- Remove the Helm release and all its components.

```
helm del --purge prometheus grafana
```

