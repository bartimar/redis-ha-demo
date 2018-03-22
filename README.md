# redis-ha-demo
Demo on Redis HA cluster deployment @ GKE

There is no persistency and no sharding. 
# tldr;
```
$ helm install stable/redis-ha
```

# customized deployment 
```
$ HELM_RELEASE_NAME=my-release
$ NAMESPACE=redis

$ helm install \
  --set replicas.servers=5,replicas.sentinels=3 \
  --name "$HELM_RELEASE_NAME" stable/redis-ha
```
for other values check the official helm chart https://github.com/kubernetes/charts/tree/master/stable/redis-ha

# test the cluster
next you can check the pod roles of the cluster
```
$ kubectl get pods -L redis-role
NAME                                            READY     STATUS    RESTARTS   AGE       REDIS-ROLE
my-release-redis-ha-sentinel-757cf877cc-5hvlw   1/1       Running   0          3m        sentinel
my-release-redis-ha-sentinel-757cf877cc-bzbbh   1/1       Running   0          3m        sentinel
my-release-redis-ha-sentinel-757cf877cc-cz8dt   1/1       Running   0          3m        sentinel
my-release-redis-ha-server-655597595-2bfkb      1/1       Running   0          33s       slave
my-release-redis-ha-server-655597595-d6jr6      1/1       Running   1          3m        slave
my-release-redis-ha-server-655597595-fklff      1/1       Running   1          3m        master
my-release-redis-ha-server-655597595-gr84b      1/1       Running   0          33s       slave
my-release-redis-ha-server-655597595-pz8t6      1/1       Running   1          3m        slave
```

execute something on the master
```
$ kubectl exec -it $(kubectl get pods -l=redis-role=master -o jsonpath="{.items[0].metadata.name}") \
  redis-cli incr mycounter   
(integer) 1
$ kubectl exec -it $(kubectl get pods -l=redis-role=master -o jsonpath="{.items[0].metadata.name}") \
  redis-cli incr mycounter  
(integer) 2
$ kubectl exec -it $(kubectl get pods -l=redis-role=master -o jsonpath="{.items[0].metadata.name}") \
  redis-cli incr mycounter   
(integer) 3
```
verify it on a slave
```
$ kubectl exec -it $(kubectl get pods -l=redis-role=slave -o jsonpath="{.items[0].metadata.name}") \
  redis-cli GET mycounter
"3"
```
# teardown the cluster
(Warning: This will delete all the data too!)
```
$ helm del --purge $HELM_RELEASE_NAME
```
