Create a new project for testing
```
oc new-project image-tool
```

Create resources for the test application
```
oc apply -f serviceaccount.yaml -f deployment.yaml -f service.yaml -f route.yaml
```

Verify the pod is running 
```
oc get pods -l app=image-tool-pvc
```

Get route and hostname
```
oc get route/image-tool-pvc -o jsonpath='{.spec.host}{"\n"}'
```

View the deployment logs
```
oc logs deployment/image-tool-pvc | head
```

Verify the message that says the application is using ephemeral storage. This means that the application container does not have a PV mounted in /var/storage.
```
 INFO in app: Using EPHEMERAL as backing storage
```

## Configuring cephfs

Now we configure persistence storage for the application (Here look for cephfs)
```
oc get storageclasses -o name
storageclass.storage.k8s.io/lso-volumeset
storageclass.storage.k8s.io/nfs-storage
storageclass.storage.k8s.io/ocs-storagecluster-ceph-rbd
storageclass.storage.k8s.io/ocs-storagecluster-ceph-rgw
storageclass.storage.k8s.io/ocs-storagecluster-cephfs
storageclass.storage.k8s.io/openshift-storage.noobaa.io
```

Now we use the deployment.yaml and the pvc.yaml found in the project and check the pods 
```
oc apply -f pvc.yaml -f deployment.yaml

oc get pods
```

Check the persistencevolume claim
```
oc describe deployment/image-tool-pvc
...output omitted...
  Containers:
   image-tool-pvc:
    Image:        quay.io/redhattraining/image-tool:latest
    Port:         5000/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:
      /var/storage from image-tool-storage (rw)
  Volumes:
   image-tool-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  image-tool
    ReadOnly:   false
...output omitted...
```

Verify that /var/storage is mounted in the container.
```
oc exec -it deployment/image-tool-pvc df -h /var/storage
Filesystem                             Size   Used   Avail   Use%   Mounted on
172.30.200.149:6789,172.30.41.15:...   1.0G      0    1.0G     0%   /var/storage
```