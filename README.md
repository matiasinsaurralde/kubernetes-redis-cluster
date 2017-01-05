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
apt-get install -y ruby vim wget redis-tools dnsutils
```

*Note:* `redis-trib` doesn't support hostnames (see [this issue](https://github.com/antirez/redis/issues/2565)), so we use `dig` to resolve our cluster IPs.

```
gem install redis
wget http://download.redis.io/redis-stable/src/redis-trib.rb
chmod +x redis-trib.rb
```

```
./redis-trib.rb create --replicas 1 \
 `dig +short redis-1.default.svc.cluster.local`:6379 \
 `dig +short redis-2.default.svc.cluster.local`:6379 \
 `dig +short redis-3.default.svc.cluster.local`:6379 \
 `dig +short redis-4.default.svc.cluster.local`:6379 \
 `dig +short redis-5.default.svc.cluster.local`:6379 \
 `dig +short redis-6.default.svc.cluster.local`:6379
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
