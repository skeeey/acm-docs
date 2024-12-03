# Auto-import-secret-exists error when importing a cluster

Your cluster import fails with an error message that reads: auto import secret exists. 

## Symptom: Auto import secret exists error when importing a cluster 

When importing a hive cluster for management, an `auto-import-secret already exists` error is displayed. 

## Resolving the problem: Auto-import-secret-exists error when importing a cluster

This problem occurs when you attempt to import a cluster that was previously managed by Red Hat Advanced Cluster Management. When this happens, the secrets conflict when you try to reimport the cluster. 

To work around this problem, complete the following steps:

1. To manually delete the existing `auto-import-secret`, run the following command on the hub cluster:

   ```
   oc delete secret auto-import-secret -n <cluster-namespace>
   ```

   Replace `cluster-namespace` with the namespace of your cluster.
2. Import your cluster again by using the procedure in [Cluster import introduction](../clusters/cluster_lifecycle/import_intro.adoc#import-intro).
