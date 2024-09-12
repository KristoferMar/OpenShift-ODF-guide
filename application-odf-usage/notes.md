Watch for the ceph pods 
```
oc get pods -n openshift-storage
```

View the OSD logs by specifying a rook-ceph-osd pod.
```
oc logs rook-ceph-mon-a-585ffd45d5-7vvlv -c mon -n openshift-storage
```

View the Rook-Ceph Operator logs by specifying a rook-ceph-operator pod.
```
oc logs rook-ceph-operator-548bcdc79f-xcgjb -c rook-ceph-operator -n openshift-storage
```

View the CephFS file storage logs by specifying a csi-cephfsplugin pod.
```
oc logs csi-cephfsplugin-722b2 -c csi-cephfsplugin -n openshift-storage
```

View the RBD plug-in logs by specifying a csi-rbdplugin pod.
```
 oc logs csi-rbdplugin-86dpc -c csi-rbdplugin -n openshift-storage
```

Ceph CLI to view information about Ceph.
```
oc exec -it \
  pod/rook-ceph-operator-548bcdc79f-xcgjb -n openshift-storage -c rook-ceph-operator -- /bin/bash
```

Obtain the cluster health with the ceph command and the health operator. The Ceph configuration is specified using the -c flag.
```
ceph -c /var/lib/rook/openshift-storage/openshift-storage.config health
```

Obtain the cluster status using the -s flag.
```
bash-4.4$ ceph -c /var/lib/rook/openshift-storage/openshift-storage.config -s
```

Obtain the cluster disk usage by using the df operator.
```
bash-4.4$ ceph -c /var/lib/rook/openshift-storage/openshift-storage.config df
```

Use the must-gather tool to obtain Ceph cluster informaton.
```
 oc adm must-gather \
  --image=registry.redhat.io/ocs4/ocs-must-gather-rhel8:v4.7 \
  --dest-dir=must-gather
```