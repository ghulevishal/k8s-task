## Monitoring with Prometheus

### Demo.

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

Now try to access the Prometheus UI by using NodePort shown above. In Prometheus UI if go to the `Status`-> `Rule`, You see `No rules defined`. Similarly, in Prometheus UI if go to `Alert` you can see `No alerting rules defined`.

Let's configure `Record Rules` and `Alert Rules`.

- Get the  [values.yaml](https://raw.githubusercontent.com/helm/charts/master/stable/prometheus/values.yaml) and modifiy as below.

- In `values.yaml` find the section of `alertmanagerFiles` and update as below.

```yaml
.
.
.
.
## alertmanager ConfigMap entries
##
alertmanagerFiles:
  alertmanager.yml:
    global: {}
    global:
      smtp_smarthost: 'smtp.gmail.com:587'
      smtp_from: 'sender@gmail.com'
    templates:
    - '/etc/alertmanager/template/*.tmpl'
    route:
      receiver: email
    receivers:
    - name: 'email'
      email_configs:
      - to: 'reciever@gmail.com'
        from: "sender@gmail.com"
        smarthost: smtp.gmail.com:587
        auth_username: "sender@gmail.com"
        auth_identity: "sender@gmail.com"
        auth_password: "************"
.
.
.
.
.
.
```

Update SMTP Host, the Sender and reciever Email-ID. Update authentication for sender. 

- Configure the `Alert Rule` in `values.yaml`, Find a `serverFiles:` section and update with your alert rules in `alerts` section. For example take a look at the following configuration

```yaml
.
.
.
.
## Prometheus server ConfigMap entries
##
serverFiles:

  ## Alerts configuration
  ## Ref: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/
  alerts: 
    groups:
      - name: k8s_alerts
        rules:
        - alert: MoreThan30Deployments
          expr: count(kube_deployment_created) >= 30
          for: 2m
          labels:
            severity: critical
          annotations:
            summary: Hey Admin!!!!! More Than 30 Deployments are running in cluster
      - name: node_alerts
        rules:
        - alert: HighNodeCPU
          expr: instance:node_cpu:avg_rate5m > 20
          for: 10s
          labels:
            severity: warning
          annotations:
            summary: High Node CPU of {{ humanize $value}}% for 1 hour
.
.
.
.
.
```

- Configure the `Record Rule` in `values.yaml`, Find a `serverFiles:` section and update with your record rules in `rules` section. For example it will look like the following configuration

```
.
.
.
serverFiles:

  ## Alerts configuration
  ## Ref: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/
  alerts: 
    groups:
      - name: k8s_alerts
        rules:
        - alert: MoreThan30Deployments
          expr: count(kube_deployment_created) >= 30
          for: 2m
          labels:
            severity: critical
          annotations:
            summary: Hey Admin!!!!! More Than 30 Deployments are running in cluster
      - name: node_alerts
        rules:
        - alert: HighNodeCPU
          expr: instance:node_cpu:avg_rate5m > 20
          for: 10s
          labels:
            severity: warning
          annotations:
            summary: High Node CPU of {{ humanize $value}}% for 1 hour

  rules: 
    groups:
      - name: kubernetes_rules
        rules:
        - record: apiserver_latency_seconds:quantile
          expr: histogram_quantile(0.99, rate(apiserver_request_latencies_bucket[5m])) / 1e+06
          labels:
            quantile: "0.99"
        - record: apiserver_latency_seconds:quantile
          expr: histogram_quantile(0.9, rate(apiserver_request_latencies_bucket[5m])) / 1e+06
          labels:
            quantile: "0.9"
        - record: apiserver_latency_seconds:quantile
          expr: histogram_quantile(0.5, rate(apiserver_request_latencies_bucket[5m])) / 1e+06
          labels:
            quantile: "0.5"
      - name: node_rules
        rules:
        - record: instance:node_cpu:avg_rate5m
          expr: 100 - avg (irate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance) * 100
        - record: instance:node_memory_usage:percentage
          expr: (node_memory_MemTotal_bytes - (node_memory_MemFree + node_memory_Cached_bytes + node_memory_Buffers_bytes)) / node_memory_MemTotal_bytes * 100
        - record: instance:root:node_filesystem_usage:percentage
          expr: (node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_free_bytes{mountpoint="/"}) / node_filesystem_size_bytes{mountpoint="/"}* 100
      - name: k8s_rules
        rules:
        - record: k8s:service:count
          expr: count(kube_service_created)
        - record: k8s:pod:count
          expr: count(kube_pod_created)
        - record: k8s:deploy:count
          expr: count(kube_deployment_created)
        - record: k8s:clusterrole:count
          expr: etcd_object_counts{job="kubernetes-apiservers",resource="clusterroles.rbac.authorization.k8s.io"}
        - record: k8s:serviceaccount:count
          expr: etcd_object_counts{job="kubernetes-apiservers",resource="serviceaccounts"}	

  prometheus.yml:
    rule_files:
      - /etc/config/rules
      - /etc/config/alerts

    scrape_configs:
      - job_name: prometheus
        static_configs:
          - targets:
            - localhost:9090
.
.
.
.
        
```

Once you update the `values.yaml` Lets now upgrade the Helm chart.

- Upgrade the Helm release.

```command
helm upgrade prometheus --set server.service.type=NodePort stable/prometheus -f ./values.yaml
```

Once you upgrade the Helm release. Access the Prometheus UI. In Prometheus UI if go to the `Status`-> `Rule`, You see new rules we have configure earlier are present there. Similarly, in Prometheus UI if you go to `Alert`, you can see that Alert rules are present there.





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

