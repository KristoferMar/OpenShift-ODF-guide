# OpenShift-OFD-guide

## Installation
1. Install the two following operators in the UI.
``` Local Storage ```
&
``` OpenShift Data Foundation ```

### Installation using CLI
#### Label all worker nodes
```
oc get nodes -l node-role.kubernetes.io/worker=
NAME       STATUS   ROLES    AGE   VERSION
worker01   Ready    worker   14d   v1.20.0+a0b09eb
worker02   Ready    worker   14d   v1.20.0+a0b09eb
worker03   Ready    worker   14d   v1.20.0+a0b09eb
```
Given the name of your notes are "worker01" etc we label them like this
```
oc label nodes -l node-role.kubernetes.io/worker= cluster.ocs.openshift.io/openshift-storage=
node/worker01 labeled
node/worker02 labeled
node/worker03 labeled
```
We now analyze the disks available on one of the workernotes
```
oc debug node/worker01 -- lsblk --paths --nodeps
```
Based on what we see here we want to go for space on /vdb and /vdc given they are not in use and have space available 

Now we create an "OperatorGroup" which defines the scope of the operators that will be deployed in our namespace. Modify and use the "lso-operatorgroup.yaml" for this.
- This is done to limit the scope of the operator to our namespace for security or resource management reasons.
```
oc apply -f lso-operatorgroup.yaml
```

Now we create the "openshift-local-storage" namespace (more info in README_LocalStorage.yaml) and change into it.
```
oc adm new-project openshift-local-storage

oc project openshift-local-storage
```

And then we install the LocalStorage operator using the lso-subscription.yaml and watch the installation
```
oc apply -f lso-subscription.yml

watch oc get -n openshift-local-storage operatorgroups,subscriptions,clusterserviceversions
```

We then apply the localvolumediscovery.yaml and get the output 
```
oc apply -f localvolumediscovery.yml

oc get localvolumediscoveryresults
```

We then verify that /dev/vdb and /dev/vdc disks were discovered on all worker notes and it should look like this 
```
oc get localvolumediscoveryresults -o custom-columns='NAME:metadata.name,PATH:status.discoveredDevices[*].path'
NAME                        PATH
discovery-result-worker01   /dev/vda1,/dev/vda2,...,/dev/vdb,/dev/vdc
discovery-result-worker02   /dev/vda1,/dev/vda2,...,/dev/vdb,/dev/vdc
discovery-result-worker03   /dev/vda1,/dev/vda2,...,/dev/vdb,/dev/vdc
```

We then create the local volume set by applying the localvolumeset.yml file and keep track 
```
oc apply -f localvolumeset.yml

oc get localvolumesets/lso-volumeset
NAME            STORAGECLASS    PROVISIONED   AGE
lso-volumeset   lso-volumeset   3             2m
```

# Openshift-storage
We create a new project called "openshift-storage" and shift to it

```
oc adm new-project openshift-storage
Created project openshift-storage

oc project openshift-storage
Now using project "openshift-storage" on server "https://api.ocp4.example.com:6443".
```

We apply the operatorgroup to the namespace
```
oc apply -f /openshift-storage/ocs-operatorgroup.yml
```

We create the osc-operator subscription and watch the installation (can take a few minutes)
```
oc apply -f /openshift-storage/ocs-subscription.yml

watch oc get -n openshift-storage operatorgroups,subscriptions,clusterserviceversions
```

We create the osc-storagecluster and watch the installation
```
oc apply -f /openshift-storage/storagecluster.yml

watch oc get -n openshift-storage storagecluster,pods
```