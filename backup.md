# ETCD backup
If etcd data is lost then this is much worse than temporary outage. <br>
without a backup  kubernetes losses entire clusters state. <br>
even though kubernetes pods may still running on worker nodes, kubernetes no longer knows about them and can not manage them correctly. <br>

### Verify your cluster uses etcd
```
kubectl get pods -n kube-system | grep etcd
```
<img width="827" height="33" alt="image" src="https://github.com/user-attachments/assets/95d601f1-82ee-4037-8062-02dfc2a44211" />

### locate the etcd certificates
```
cat /etc/kubernetes/manifests/etcd.yaml
```

you will find
```
--cert-file=/etc/kubernetes/pki/etcd/server.crt
--key-file=/etc/kubernetes/pki/etcd/server.key
--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```
### Take backup of `/etc/kubernetes` directory.
```
tar -cvzf etc.kubernetes.tar.gz /etc/kubernetes
```
### Take snapshot of the etcd
```
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%F-%H%M).db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key
```

### Store the backup on safe location.
