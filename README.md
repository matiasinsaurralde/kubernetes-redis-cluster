# Kubernetes Redis Cluster

### Create Disks

```
gcloud compute disks create --size=10GB \
  'redis-1' 'redis-2' 'redis-3' \
  'redis-4' 'redis-5' 'redis-6'
```

### Create Redis Cluster Configuration

```
kubectl create configmap redis-conf --from-file=redis.conf
```

### Create Redis Nodes

```
kubectl create -f replicasets
```

### Create Redis Services

```
kubectl create -f services
```

### Connect Nodes

```
kubectl run -i --tty ubuntu --image=ubuntu \
  --restart=Never /bin/bash
```

```
apt-get update
apt-get install -y python2.7 python-pip redis-tools dnsutils
```

*Note:* `redis-trib` doesn't support hostnames (see [this issue](https://github.com/antirez/redis/issues/2565)), so we use `dig` to resolve our cluster IPs.

```
pip install redis-trib
```

```
redis-trib.py create `dig +short redis-1.default.svc.cluster.local`:6379 `dig +short redis-2.default.svc.cluster.local`:6379 `dig +short redis-3.default.svc.cluster.local`:6379
redis-trib.py replicate --master-addr `dig +short redis-1.default.svc.cluster.local`:6379 --slave-addr `dig +short redis-4.default.svc.cluster.local`:6379
redis-trib.py replicate --master-addr `dig +short redis-2.default.svc.cluster.local`:6379 --slave-addr `dig +short redis-5.default.svc.cluster.local`:6379
redis-trib.py replicate --master-addr `dig +short redis-3.default.svc.cluster.local`:6379 --slave-addr `dig +short redis-6.default.svc.cluster.local`:6379
```

### Add a new node

```
gcloud compute disks create --size=10GB 'redis-7'
```

```
kubectl create -f replicaset/redis-7.yaml
```

```
kubectl create -f services/redis-7.yaml
```

```
CLUSTER MEET 10.131.242.7 6379
```
