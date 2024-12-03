# Troubleshooting an offline cluster

There are a few common causes for a cluster showing an offline status. 

## Symptom: Cluster status is offline

After you complete the procedure for creating a cluster, you cannot access it from the Red Hat Advanced Cluster Management console, and it shows a status of `offline`.

## Resolving the problem: Cluster status is offline

1. Determine if the managed cluster is available. You can check this in the _Clusters_ area of the Red Hat Advanced Cluster Management console. 

+
If it is not available, try restarting the managed cluster.

1. If the managed cluster status is still offline, complete the following steps:
   1. Run the `oc get managedcluster <cluster_name> -o yaml` command on the hub cluster. Replace `<cluster_name>` with the name of your cluster.
   2. Find the `status.conditions` section.
   3. Check the messages for `type: ManagedClusterConditionAvailable` and resolve any problems.
