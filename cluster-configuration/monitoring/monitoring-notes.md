Check for storageclasses and look for the "ocs-storagecluster-ceph-rbd"
```
oc get storageclasses -o name
storageclass.storage.k8s.io/lso-volumeset
storageclass.storage.k8s.io/nfs-storage
storageclass.storage.k8s.io/ocs-storagecluster-ceph-rbd
storageclass.storage.k8s.io/ocs-storagecluster-ceph-rgw
storageclass.storage.k8s.io/ocs-storagecluster-cephfs
storageclass.storage.k8s.io/openshift-storage.noobaa.io
```

Verify that prometheus-k8s is using the right configuration
```
# Prometheus container volume mount spec:
oc get statefulset/prometheus-k8s \
  -n openshift-monitoring \
  -o jsonpath='{.spec.template.spec.containers}' | \
jq '.[] | select(.name == "prometheus") | .volumeMounts[] | select(.name == "prometheus-k8s-db")'
{
  "mountPath": "/prometheus",
  "name": "prometheus-k8s-db"
}

# Prometheus volume mount type:
oc get statefulset/prometheus-k8s \
  -n openshift-monitoring \
  -o jsonpath='{.spec.template.spec.volumes}' | \
  jq '.[] | select(.name == "prometheus-k8s-db")'
{
  "emptyDir": {},
  "name": "prometheus-k8s-db"
}
```

Verify the disk space in the emptyDir volume mounted in /prometheus
```
oc exec -n openshift-monitoring statefulset/prometheus-k8s -c prometheus -- df -h /prometheus
```

Verify that the alertmanager-main stateful set in the openshift-monitoring namespace uses ephemeral storage at the /alertmanager mount point.
```
# AlertManager container volume mount spec:
oc get statefulset/alertmanager-main \
  -n openshift-monitoring \
  -o jsonpath='{.spec.template.spec.containers}' | \
jq '.[] | select(.name == "alertmanager") | .volumeMounts[] | select(.name == "alertmanager-main-db")'
{
  "mountPath": "/alertmanager",
  "name": "alertmanager-main-db"
}

# AlertManager volume mount type:
oc get statefulset/alertmanager-main \
  -n openshift-monitoring \
  -o jsonpath='{.spec.template.spec.volumes}' | \
  jq '.[] | select(.name == "alertmanager-main-db")'
{
  "emptyDir": {},
  "name": "alertmanager-main-db"
}
```

Verify the disk space in the emptyDir volume mounted in /prometheus.
```
oc exec -n openshift-monitoring statefulset/alertmanager-main -c alertmanager -- df -h /alertmanager
```

Now we create and use a cluster-monitoring-config file and we call it metrics-storage.yaml
```
oc create -n openshift-monitoring configmap cluster-monitoring-config --from-file config.yaml=metrics-storage.yml
```

Verify that both Prometheus and Alertmanager use block storage from OpenShift Data Foundation.
```
watch oc get statefulsets -n openshift-monitoring
```

Verify the PVCs that are used to save the monitoring data are created.
```
oc get persistentvolumeclaims -n openshift-monitoring
```

We then need to verify that the prometheus-k8s stateful set in the openshift-monitoring namespace mounts a block device to the /prometheus mount point
```
oc exec -n openshift-monitoring statefulset/prometheus-k8s -c prometheus -- df -h /prometheus
```

We also verify that the alertmanager-main stateful set in the openshift-monitoring namespace mounts a block device to the /alertmanager mount point.
```
oc exec -n openshift-monitoring statefulset/alertmanager-main -c alertmanager -- df -h /alertmanager
```