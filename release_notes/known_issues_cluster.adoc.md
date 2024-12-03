# Cluster management known issues and limitations

Review the known issues for cluster management with Red Hat Advanced Cluster Management. The following list contains known issues and limitations for this release, or known issues that continued from the previous release.

For Cluster lifecycle with the multicluster engine for Kubernetes operator known issues, see [Cluster lifecycle known issues and limitations](../clusters/release_notes/mce_known_issues.adoc#known-issues-mce) in the multicluster engine operator documentation.

## Hub cluster communication limitations

The following limitations occur if the hub cluster is not able to reach or communicate with the managed cluster:

* You cannot create a new managed cluster by using the console. You are still able to import a managed cluster manually by using the command line interface or by using the **Run import commands manually** option in the console.
* If you deploy an Application or ApplicationSet by using the console, or if you import a managed cluster into ArgoCD, the hub cluster ArgoCD controller calls the managed cluster API server. You can use AppSub or the ArgoCD pull model to work around the issue.
* The console page for pod logs does not work, and an error message that resembles the following appears:

  ```
  Error querying resource logs:
  Service unavailable
  ```

## The local-cluster might not be automatically recreated

If the local-cluster is deleted while `disableHubSelfManagement` is set to `false`, the local-cluster is recreated by the `MulticlusterHub` operator. After you detach a local-cluster, the local-cluster might not be automatically recreated. 

* To resolve this issue, modify a resource that is watched by the `MulticlusterHub` operator. See the following example:

+
```
oc delete deployment multiclusterhub-repo -n <namespace>
```

* To properly detach the local-cluster, set the `disableHubSelfManagement` to true in the `MultiClusterHub`.

## Local-cluster status offline after reimporting with a different name

When you accidentally try to reimport the cluster named `local-cluster` as a cluster with a different name, the status for `local-cluster` and for the reimported cluster display `offline`.

To recover from this case, complete the following steps:

1. Run the following command on the hub cluster to edit the setting for self-management of the hub cluster temporarily:

   ```
   oc edit mch -n open-cluster-management multiclusterhub
   ```
2. Add the setting `spec.disableSelfManagement=true`.
3. Run the following command on the hub cluster to delete and redeploy the local-cluster:

   ```
   oc delete managedcluster local-cluster
   ```
4. Enter the following command to remove the `local-cluster` management setting: 

   ```
   oc edit mch -n open-cluster-management multiclusterhub
   ```
5. Remove `spec.disableSelfManagement=true` that you previously added.

## Hub cluster and managed clusters clock not synced

Hub cluster and manage cluster time might become out-of-sync, displaying in the console `unknown` and eventually `available` within a few minutes. Ensure that the OpenShift Container Platform hub cluster time is configured correctly. See [Customizing nodes](https://docs.redhat.com/en/documentation/openshift_container_platform/4.14/html/installation_configuration/installing-customizing).
