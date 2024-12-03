# Troubleshooting a managed cluster import failure

If your cluster import fails, there are a few steps that you can take to determine why the cluster import failed.  

## Symptom: Imported cluster not available

After you complete the procedure for importing a cluster, you cannot access it from the Red Hat Advanced Cluster Management for Kubernetes console.

## Resolving the problem: Imported cluster not available

There can be a few reasons why an imported cluster is not available after an attempt to import it. If the cluster import fails, complete the following steps, until you find the reason for the failed import:

1. On the Red Hat Advanced Cluster Management hub cluster, run the following command to ensure that the Red Hat Advanced Cluster Management import controller is running. 

   ```
   kubectl -n multicluster-engine get pods -l app=managedcluster-import-controller-v2
   ```

   You should see two pods that are running. If either of the pods is not running, run the following command to view the log to determine the reason:

   ```
   kubectl -n multicluster-engine logs -l app=managedcluster-import-controller-v2 --tail=-1
   ```
2. On the Red Hat Advanced Cluster Management hub cluster, run the following command to determine if the managed cluster import secret was generated successfully by the Red Hat Advanced Cluster Management import controller:

   ```
   kubectl -n <managed_cluster_name> get secrets <managed_cluster_name>-import
   ```

   If the import secret does not exist, run the following command to view the log entries for the import controller and determine why it was not created:

   ```
   kubectl -n multicluster-engine logs -l app=managedcluster-import-controller-v2 --tail=-1 | grep importconfig-controller
   ```
3. On the Red Hat Advanced Cluster Management hub cluster, if your managed cluster is `local-cluster`, provisioned by Hive, or has an auto-import secret, run the following command to check the import status of the managed cluster.

   ```
   kubectl get managedcluster <managed_cluster_name> -o=jsonpath='{range .status.conditions[*]}{.type}{"\t"}{.status}{"\t"}{.message}{"\n"}{end}' | grep ManagedClusterImportSucceeded
   ```

   If the condition `ManagedClusterImportSucceeded` is not `true`, the result of the command indicates the reason for the failure.
4. Check the Klusterlet status of the managed cluster for a degraded condition. See [Troubleshooting Klusterlet with degraded conditions](../troubleshooting/trouble_klusterlet_degraded.adoc#troubleshooting-klusterlet-with-degraded-conditions) to find the reason that the Klusterlet is degraded.
