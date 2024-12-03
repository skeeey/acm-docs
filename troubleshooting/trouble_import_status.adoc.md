# Troubleshooting cluster with pending import status

If you receive _Pending import_ continually on the console of your cluster, follow the procedure to troubleshoot the problem.

## Symptom: Cluster with pending import status

After importing a cluster by using the Red Hat Advanced Cluster Management console, the cluster appears in the console with a status of _Pending import_.

## Identifying the problem: Cluster with pending import status

1. Run the following command on the managed cluster to view the Kubernetes pod names that are having the issue:

   ```
   kubectl get pod -n open-cluster-management-agent | grep klusterlet-registration-agent
   ```
2. Run the following command on the managed cluster to find the log entry for the error:

   ```
   kubectl logs <registration_agent_pod> -n open-cluster-management-agent
   ```

   Replace _registration_agent_pod_ with the pod name that you identified in step 1.
3. Search the returned results for text that indicates there was a networking connectivity problem.
Example includes: `no such host`.

## Resolving the problem: Cluster with pending import status

1. Retrieve the port number that is having the problem by entering the following command on the hub cluster:

   ```
   oc get infrastructure cluster -o yaml | grep apiServerURL
   ```
2. Ensure that the hostname from the managed cluster can be resolved, and that outbound connectivity to the host and port is occurring.

   If the communication cannot be established by the managed cluster, the cluster import is not complete.
   The cluster status for the managed cluster is _Pending import_.
