# Troubleshooting ocm-controller errors after Red Hat Advanced Cluster Management upgrade

After you upgrade from 2.7.x to 2.8.x and then to 2.9.0, the `ocm-controller` of the `multicluster-engine` namespace crashes.

## Symptom: Troubleshooting ocm-controller errors after Red Hat Advanced Cluster Management upgrade 

After you attempt to list `ManagedClusterSet` and `ManagedClusterSetBinding` custom resource definitions, the following error message appears:

```bash
Error from server: request to convert CR from an invalid group/version: cluster.open-cluster-management.io/v1beta1
```

The previous message indicates that the migration of `ManagedClusterSets` and `ManagedClusterSetBindings` custom resource definitions from `v1beta1` to `v1beta2` failed.

## Resolving the problem: Troubleshooting ocm-controller errors after Red Hat Advanced Cluster Management upgrade

To resolve this error, you must initiate the API migration manually. Complete the following steps:

1. Revert the `cluster-manager` to a previous release. 
   1. Pause the `multiclusterengine` with the following command:

      ```bash
      oc annotate mce multiclusterengine pause=true
      ```
      +	
   2. Run the following commands to replace the image of the `cluster-manager` deployment with a previous version:

      ```bash
      oc patch deployment cluster-manager -n multicluster-engine -p \  '{"spec":{"template":{"spec":{"containers":[{"name":"registration-operator","image":"registry.redhat.io/multicluster-engine/registration-operator-rhel8@sha256:35999c3a1022d908b6fe30aa9b85878e666392dbbd685e9f3edcb83e3336d19f"}]}}}}'
      export ORIGIN_REGISTRATION_IMAGE=$(oc get clustermanager cluster-manager -o jsonpath='{.spec.registrationImagePullSpec}')
      ```
      +	
   3. Replace the registration image reference in the `ClusterManager` resource with a previous version. Run the following command:

      ```bash
      oc patch clustermanager cluster-manager --type='json' -p='[{"op": "replace", "path": "/spec/registrationImagePullSpec", "value": "registry.redhat.io/multicluster-engine/registration-rhel8@sha256:a3c22aa4326859d75986bf24322068f0aff2103cccc06e1001faaf79b9390515"}]' 
      ```
2. Run the following commands to revert the `ManagedClusterSets` and `ManagedClusterSetBindings` custom resource definitions to a previous release:

   ```bash
   oc annotate crds managedclustersets.cluster.open-cluster-management.io operator.open-cluster-management.io/version-
   oc annotate crds  managedclustersetbindings.cluster.open-cluster-management.io operator.open-cluster-management.io/version- 
   ```
3. Restart the `cluster-manager` and wait for the custom resource definitions to be recreated. Run the following commands:

   ```bash
   oc -n multicluster-engine delete pods -l app=cluster-manager
   oc wait crds managedclustersets.cluster.open-cluster-management.io --for=jsonpath="{.metadata.annotations['operator\.open-cluster-management\.io/version']}"="2.3.3" --timeout=120s
   oc wait crds managedclustersetbindings.cluster.open-cluster-management.io --for=jsonpath="{.metadata.annotations['operator\.open-cluster-management\.io/version']}"="2.3.3" --timeout=120s 
   ```
4. Start the storage version migration with the following commands:

   ```bash
   oc patch StorageVersionMigration managedclustersets.cluster.open-cluster-management.io --type='json' -p='[{"op":"replace", "path":"/spec/resource/version", "value":"v1beta1"}]'
   oc patch StorageVersionMigration managedclustersets.cluster.open-cluster-management.io --type='json' --subresource status -p='[{"op":"remove", "path":"/status/conditions"}]'
   oc patch StorageVersionMigration managedclustersetbindings.cluster.open-cluster-management.io --type='json' -p='[{"op":"replace", "path":"/spec/resource/version", "value":"v1beta1"}]'
   oc patch StorageVersionMigration managedclustersetbindings.cluster.open-cluster-management.io --type='json' --subresource status -p='[{"op":"remove", "path":"/status/conditions"}]' 
   ```
5. Run the following command to wait for the migration to complete:

   ```bash
   oc wait storageversionmigration managedclustersets.cluster.open-cluster-management.io --for=condition=Succeeded --timeout=120s 
   oc wait storageversionmigration managedclustersetbindings.cluster.open-cluster-management.io --for=condition=Succeeded --timeout=120s
   ```
6. Restore the `cluster-manager` back to Red Hat Advanced Cluster Management 2.12. It might take several minutes. Run the following command:

   ```bash
   oc annotate mce multiclusterengine pause-
   oc patch clustermanager cluster-manager --type='json' -p='[{"op": "replace", "path": "/spec/registrationImagePullSpec", "value": "'$ORIGIN_REGISTRATION_IMAGE'"}]' 
   ```

### Verification 

To verify that Red Hat Advanced Cluster Management is recovered run the following commands:

```bash
oc get managedclusterset
oc get managedclustersetbinding -A 
```

After running the commands, the `ManagedClusterSets` and `ManagedClusterSetBindings` resources are listed without error messages.
