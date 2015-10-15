# Visit Etcd Service with Http Support

## Method 1

```
etcdctl -C http://10.1.88.70:2379 member list
etcdctl -C http://10.1.88.70:2379 ls /
etcdctl -C http://10.1.88.70:4001,http://10.1.88.71:4001,http://10.1.88.72:4001 ls /
```

you can get etcd members with the command below

```
etcdctl memeber list
```

## Method 2

use curl command with the same purpose as #method 1

```
curl -L http://127.0.0.1:4001/v2/keys/
curl -L http://127.0.0.1:4001/v2/keys/coreos.com
curl -L http://127.0.0.1:4001/v2/keys/?recursive=true
curl -L http://127.0.0.1:4001/v2/stats/leader
curl -L http://127.0.0.1:4001/v2/stats/self
curl -L http://127.0.0.1:4001/v2/stats/store
```

## Reference

https://www.digitalocean.com/community/tutorials/how-to-use-etcdctl-and-etcd-coreos-s-distributed-key-value-store