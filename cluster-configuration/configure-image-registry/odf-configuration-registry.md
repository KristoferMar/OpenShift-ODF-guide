# Storage Classes Provided by Red Hat OpenShift Data Foundation

ocs-storagecluster-ceph-rbd : This class supports block storage devices, primarily for database workloads.

ocs-storagecluster-cephfs	: This class provides shared and distributed file system data services, primarily used for software development, messaging, and data aggregation workloads.

openshift-storage.noobaa.io	: This class provides multicloud object storage through the MCG.

ocs-storagecluster-ceph-rgw	: This class provides on-premises object storage.

## Notes

We want to make sure the storageclass from noobaa.io is pressent 

```
oc get storageclasses -o name
storageclass.storage.k8s.io/lso-volumeset
storageclass.storage.k8s.io/nfs-storage
storageclass.storage.k8s.io/ocs-storagecluster-ceph-rbd
storageclass.storage.k8s.io/ocs-storagecluster-ceph-rgw
storageclass.storage.k8s.io/ocs-storagecluster-cephfs
storageclass.storage.k8s.io/openshift-storage.noobaa.io
```

Run the obc-registry.yaml and check the result
```
oc apply -f obc-registry.yml

oc get -n openshift-image-registry objectbucketclaim/noobaa-registry
```

Create a new generic secret containing the credentials needed to access and manage the OBC.
```
oc get secrets -l app=noobaa -n openshift-image-registry

oc extract secret/noobaa-registry -n openshift-image-registry
```

We then create the image-registry-private-configuratom
```
oc create secret generic \
  image-registry-private-configuration-user \
  --from-literal=REGISTRY_STORAGE_S3_ACCESSKEY="$(cat AWS_ACCESS_KEY_ID)" \
  --from-literal=REGISTRY_STORAGE_S3_SECRETKEY="$(cat AWS_SECRET_ACCESS_KEY)" \
  -n openshift-image-registry
```

We configure the internal storage to use the noobaa-registry
```
oc get -n openshift-image-registry objectbucketclaim/noobaa-registry -o jsonpath='{.spec.bucketName}{"\n"}'
```

We Identify the URL for the s3 route in the openshift-storage namespace.
```
oc get route/s3 -n openshift-storage -o jsonpath='{.spec.host}{"\n"}'
```

We apply a patch to the image registry configuration. This will change the storage type from PVC to OBC. Then we confirm at last.
```
oc patch configs.imageregistry/cluster --type=merge --patch-file=imageregistry-patch.yaml

oc get configs.imageregistry/cluster -o jsonpath='{.spec.storage}' | jq .
```

We can then monitor the deployment if image-registry
```
watch oc get pods -n openshift-image-registry -l docker-registry=default
```