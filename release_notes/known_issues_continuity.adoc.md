# Business continuity known issues

Review the known issues for Red Hat Advanced Cluster Management for Kubernetes. The following list contains known issues for this release, or known issues that continued from the previous release. 

For your Red Hat OpenShift Container Platform cluster, see [OpenShift Container Platform known issues](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/release_notes/#ocp-4-15-known-issues). 

For more about deprecations and removals, see [Deprecations and removals for Red Hat Advanced Cluster Management](../release_notes/acm_deprecate_remove.adoc#deprecations-removals-acm).

## Backup and restore known issues

Backup and restore known issues and limitations are listed here, along with workarounds if they are available.

### The _open-cluster-management-backup_ namespace is stuck in the _Terminating_ state

When the cluster-backup component is disabled on the `MultiClusterHub` resource, the `open-cluster-management-backup` namespace is stuck in the `Terminating` state if you have a Velero restore resource created by a Red Hat Advanced Cluster Management restore operation.

The `Terminating` state is a result of the Velero restore resources waiting on the `restores.velero.io/external-resources-finalizer` to complete. To workaround this issue, complete the following steps:

1. Delete all Red Hat Advanced Cluster Management restore resources and wait for the Velero restore to be cleaned up before you disable the cluster backup option on the `MultiClusterHub` resource. 
2. If your `open-cluster-management-backup` namespace is already stuck in the `Terminating` state, edit all the Velero restore resources and remove the finalizers. 
3. Allow the Velero resources to delete the namespaces and resources. 

### Bare metal hub resource no longer backed up by the managed clusters backup

If the resources for the bare metal cluster are backed up and restored to a secondary hub cluster by using the Red Hat Advanced Cluster Management back up and restore feature, the managed cluster reinstalls on the nodes, which destroys the existing managed cluster. 

**Note:** This only affects bare metal clusters that were deployed by using zero touch provisioning, meaning that they have `BareMetalHost` resources that manage powering on and off bare metal nodes and attaching virtual media for booting. If a `BareMetalHost` resource was not used in the deployment of the managed cluster, there is no negative impact.

To work around this issue, the `BareMetalHost` resources on the primary hub cluster are no longer backed up with the managed cluster backup.

If you have a different use case and want the managed `BareMetalHost` resources on the primary hub cluster to be backed up, add the following backup label to the `BareMetalHost` resources on the primary hub cluster: `cluster.open-cluster-management.io/backup`.

To learn more about using this backup label to backup generic resources, see the topic, [Resources that are backed up](../business_continuity/backup_restore/backup_arch.adoc#resources-that-are-backed-up). 
 

### Velero restore limitations

A new hub cluster can have a different configuration than the active hub cluster if the new hub cluster, where the data is restored, has user-created resources. For example, this can include an existing policy that was created on the new hub cluster before the backup data is restored on the new hub cluster.

Velero skips existing resources if they are not part of the restored backup, so the policy on the new hub cluster remains unchanged, resulting in a different configuration between the new hub cluster and active hub cluster.

To address this limitation, the cluster backup and restore operator runs a post restore operation to clean up the resources created by the user or a different restore operation when a `restore.cluster.open-cluster-management.io` resource is created.

For more information, see the [Cleaning the hub cluster after restore](../business_continuity/backup_restore/backup_restore.adoc#clean-hub-restore) topic. 

### Passive configurations do not display managed clusters

Managed clusters are only displayed when the activation data is restored on the passive hub cluster.

### Managed cluster resource not restored

When you restore the settings for the `local-cluster` managed cluster resource and overwrite the `local-cluster` data on a new hub cluster, the settings are misconfigured. Content from the previous hub cluster `local-cluster` is not backed up because the resource contains `local-cluster` specific information, such as the cluster URL details.

You must manually apply any configuration changes that are related to the `local-cluster` resource on the restored cluster. See _Prepare the new hub cluster_ in the [Installing the backup and restore operator](../business_continuity/backup_restore/backup_install.adoc#dr4hub-install-backup-and-restore) topic.

### Restored Hive managed clusters might not be able to connect with the new hub cluster

When you restore the backup of the changed or rotated certificate of authority (CA) for the Hive managed cluster, on a new hub cluster, the managed cluster fails to connect to the new hub cluster. The connection fails because the `admin` `kubeconfig` secret for this managed cluster, available with the backup, is no longer valid. 

You must manually update the restored `admin` `kubeconfig` secret of the managed cluster on the new hub cluster.

### Imported managed clusters show a _Pending Import_ status

Managed clusters that are manually imported on the primary hub cluster show a `Pending Import` status when the activation data is restored on the passive hub cluster. For more information, see [Connecting clusters by using a Managed Service Account](../business_continuity/backup_restore/backup_msa.adoc#auto-connect-clusters-msa).

### The _appliedmanifestwork_ is not removed from managed clusters after restoring the hub cluster

When the hub cluster data is restored on the new hub cluster, the `appliedmanifestwork` is not removed from managed clusters that have a placement rule for an application subscription that is not a fixed cluster set.

See the following example of a placement rule for an application subscription that is not a fixed cluster set:

```yaml
spec:
  clusterReplicas: 1
  clusterSelector:
    matchLabels:
      environment: dev
```

As a result, the application is orphaned when the managed cluster is detached from the restored hub cluster.

To avoid the issue, specify a fixed cluster set in the placement rule. See the following example:

```yaml
spec:
  clusterSelector:
    matchLabels:
      environment: dev
```

You can also delete the remaining `appliedmanifestwork` manually by running the folowing command:

```
oc delete appliedmanifestwork <the-left-appliedmanifestwork-name>
```

### The _appliedmanifestwork_ not removed and _agentID_ is missing in the specification

When you are using Red Hat Advanced Cluster Management 2.6 as your primary hub cluster, but your restore hub cluster is on version 2.7 or later, the `agentID` is missing in the specification of `appliedmanifestworks` because the field is introduced in the 2.7 release. This results in the extra `appliedmanifestworks` for the primary hub on the managed cluster.

To avoid the issue, upgrade the primary hub cluster to Red Hat Advanced Cluster Management 2.7, then restore the backup on a new hub cluster.

Fix the managed clusters by setting the `spec.agentID` manually for each `appliedmanifestwork`.

1. Run the following command to get the `agentID`: 

   ```
   oc get klusterlet klusterlet -o jsonpath='{.metadata.uid}'
   ```
2. Run the following command to set the `spec.agentID` for each `appliedmanifestwork`:

   ```
   oc patch appliedmanifestwork <appliedmanifestwork_name> --type=merge -p '{"spec":{"agentID": "'$AGENT_ID'"}}'  
   ```

### The _managed-serviceaccount_ add-on status shows _Unknown_

The managed cluster `appliedmanifestwork` `addon-managed-serviceaccount-deploy` is removed from the imported managed cluster if you are using the Managed Service Account without enabling it on the multicluster engine for Kubernetes operator resource of the new hub cluster.

The managed cluster is still imported to the new hub cluster, but 
the `managed-serviceaccount` add-on status shows `Unknown`.
 
You can recover the `managed-serviceaccount` add-on after enabling the Managed Service Account in the multicluster engine operator resource. See [Enabling automatic import](../business_continuity/backup_restore/backup_msa.adoc#enabling-auto-import) to learn how to enable the Managed Service Account.

## Volsync known issues

### Manual removal of the VolSync CSV required on managed cluster when removing the add-on

When you remove the VolSync `ManagedClusterAddOn` from the hub cluster, it removes the VolSync operator subscription on the managed cluster but does not remove the cluster service version (CSV). To remove the CSV from the managed clusters, run the following command on each managed cluster from which you are removing VolSync:

```
oc delete csv -n openshift-operators volsync-product.v0.6.0
```

### Restoring the connection of a managed cluster with custom CA certificates to its restored hub cluster might fail

After you restore the backup of a hub cluster that manages a cluster with custom CA certificates, the connection between the managed cluster and the hub cluster might fail. This is because the CA certificate was not backed up on the restored hub cluster. To restore the connection, copy the custom CA certificate information that is in the namespace of your managed cluster to the `<managed_cluster>-admin-kubeconfig` secret on the restored hub cluster. 

**Note:** If you copy this CA certificate to the hub cluster before creating the backup copy, the backup copy includes the secret information. When you use the backup copy to restore in the future, the connection between the hub cluster and managed cluster automatically completes.
