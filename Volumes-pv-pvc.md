### 1.Empty directory Volumes:
- delete when the pod is removed.
- good for temparary data.

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-example
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: /data
          name: my-volume
  volumes:
    - name: my-volume
      emptyDir: {}
```

### 2. Hostpath volume:
- mount directory or file system from the host node.<br>
- Good for debugging, but not recommended for production.<br>
```
volumes:
  - name: host-volume
    hostPath:
      path: /tmp/data
      type: Directory
```

### 3.Persistent volumes (PV) and Persistent volume claims (PVC):
- Used for durable storage e.g. for databases.

## PV - Piece of storage <br>
- Created by admin or dynamically by kubernetes<br>
- Exists independently of pods <br>
- like physical pendrive attached with cluster.<br>

## PVC:<br>
- A request of storage made by user or pod.<br>
- You specify how much storage you want and which type.<br>
- like i need 5 GB of storage.<br>

Once kubernetes finds a matching PV, it binds the PVC to it.<br>

