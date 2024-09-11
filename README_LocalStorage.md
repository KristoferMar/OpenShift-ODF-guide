# Notes

openshift-local-storage
- It is a default namespace within openshift

When we install the Local Storage Operator we want to do it to the namespace called openshift-local-storage, the operator itself is responsible for managing and provisioning local storage resources (such as PersistentVolumes) at the cluster level. These resources can then be accessed and utilized by pods in any namespace, including your personal-test-namespace namespace.

