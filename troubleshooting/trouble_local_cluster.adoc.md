# Troubleshooting local cluster not selected with placement rule

The managed clusters are selected with a placement rule, but the `local-cluster`, which is a hub cluster that is also managed, is not selected. The placement rule user is not granted permission to get the `managedcluster` resources in the `local-cluster` namespace.

## Symptom: Troubleshooting local cluster not selected as a managed cluster

All managed clusters are selected with a placement rule, but the `local-cluster` is not. The placement rule user is not granted permission to get the `managedcluster` resources in the `local-cluster` namespace.

## Resolving the problem: Troubleshooting local cluster not selected as a managed cluster

**Deprecated:** `PlacementRule`

To resolve this issue, you need to grant the `managedcluster` administrative permission in the `local-cluster` namespace. Complete the following steps:

1. Confirm that the list of managed clusters does include `local-cluster`, and that the placement rule `decisions` list does not display the `local-cluster`. Run the following command and view the results:

   ```
   % oc get managedclusters 
   ```

   See in the sample output that `local-cluster` is joined, but it is not in the YAML for `PlacementRule`:

   ```
   NAME            HUB ACCEPTED   MANAGED CLUSTER URLS   JOINED   AVAILABLE   AGE
   local-cluster   true                                  True     True        56d
   cluster1        true                                  True     True        16h
   ```

+
```yaml
apiVersion: apps.open-cluster-management.io/v1
kind: PlacementRule
metadata:
  name: all-ready-clusters
  namespace: default
spec:
  clusterSelector: {}
status:
  decisions:
  - clusterName: cluster1
    clusterNamespace: cluster1
```

1. Create a `Role` in your YAML file to grant the `managedcluster` administrative permission in the `local-cluster` namespace. See the following example:

+
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: managedcluster-admin-user-zisis
  namespace: local-cluster
rules:
- apiGroups:
  - cluster.open-cluster-management.io
  resources:
  - managedclusters
  verbs:
  - get
```

1. Create a `RoleBinding` resource to grant the placement rule user access to the `local-cluster` namespace. See the following example:

+
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: managedcluster-admin-user-zisis
  namespace: local-cluster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: managedcluster-admin-user-zisis
  namespace: local-cluster
subjects:
- kind: User
  name: zisis
  apiGroup: rbac.authorization.k8s.io
```
