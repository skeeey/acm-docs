# Troubleshooting cluster in console with pending or failed status

If you observe _Pending_ status or _Failed_ status in the console for a cluster you created, follow the procedure to troubleshoot the problem.

## Symptom: Cluster in console with pending or failed status

After creating a new cluster by using the Red Hat Advanced Cluster Management for Kubernetes console, the cluster does not progress beyond the status of _Pending_ or displays _Failed_ status.

## Identifying the problem: Cluster in console with pending or failed status

If the cluster displays _Failed_ status, navigate to the details page for the cluster and follow the link to the logs provided. If no logs are found or the cluster displays _Pending_ status, continue with the following procedure to check for logs:

* Procedure 1
  1. Run the following command on the hub cluster to view the names of the Kubernetes pods that were created in the namespace for the new cluster:

     ```
     oc get pod -n <new_cluster_name>
     ```

     Replace `_new_cluster_name_` with the name of the cluster that you created.
  2. If no pod that contains the string `provision` in the name is listed, continue with Procedure 2.
  If there is a pod with `provision` in the title, run the following command on the hub cluster to view the logs of that pod:

     ```
     oc logs <new_cluster_name_provision_pod_name> -n <new_cluster_name> -c hive
     ```

     Replace `new_cluster_name_provision_pod_name` with the name of the cluster that you created, followed by the pod name that contains `provision`.
  3. Search for errors in the logs that might explain the cause of the problem.
* Procedure 2

  If there is not a pod with `provision` in its name, the problem occurred earlier in the process. Complete the following procedure to view the logs:
  1. Run the following command on the hub cluster:

     ```
     oc describe clusterdeployments -n <new_cluster_name>
     ```

     Replace `new_cluster_name` with the name of the cluster that you created.
     For more information about cluster installation logs, see [Gathering installation logs](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.4/html/installing/installing-gather-logs) in the Red Hat OpenShift documentation. 
  2. See if there is additional information about the problem in the _Status.Conditions.Message_ and _Status.Conditions.Reason_ entries of the resource.
   
  ## Resolving the problem: Cluster in console with pending or failed status

After you identify the errors in the logs, determine how to resolve the errors before you destroy the cluster and create it again.

The following example provides a possible log error of selecting an unsupported zone, and the actions that are required to resolve it:

```
No subnets provided for zones
```

When you created your cluster, you selected one or more zones within a region that are not supported. Complete one of the following actions when you recreate your cluster to resolve the issue:

* Select a different zone within the region.
* Omit the zone that does not provide the support, if you have other zones listed.
* Select a different region for your cluster.

After determining the issues from the log, destroy the cluster and recreate it. 

See [Creating clusters](../clusters/cluster_lifecycle/create_intro.adoc#create-intro) for more information about creating a cluster.
