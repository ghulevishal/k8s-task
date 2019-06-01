## Logging

- Create a new namespace.

```command
kubectl create ns kube-logging
```


- Apply the manifiest.

```command
kubectl -n kube-logging apply -f efk/.
```

- Get the list of pods.

```command
kubectl get pod -n kube-logging
```
```
NAME                     READY   STATUS    RESTARTS   AGE
es-cluster-0             1/1     Running   0          58s
es-cluster-1             1/1     Running   0          42s
es-cluster-2             1/1     Running   0          26s
fluentd-d6l2w            1/1     Running   0          57s
kibana-bd6f49775-4cksr   1/1     Running   0          57s
```

- Get list of service

```command
kubectl get svc -n kube-logging
```
```
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
elasticsearch   ClusterIP   None             <none>        9200/TCP,9300/TCP   36s
kibana          NodePort    10.110.133.131   <none>        5601:31547/TCP      35s
```

Try to access kibana using public ip and nodeport.

- In Kibana GUI, Click on **Discover** in the left-hand navigation menu.

- We will just use the `logstash-*` wildcard pattern to capture all the log data in our Elasticsearch cluster. Enter `logstash-*` in the text box and click on **Next step**.

- In next page In the dropdown, select the `@timestamp` field, and hit **Create index pattern**.

- Now, hit **Discover** in the left hand navigation menu. You should see a histogram graph and some recent log entries.
