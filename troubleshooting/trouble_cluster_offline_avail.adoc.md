# Troubleshooting cluster status changing from offline to available

The status of the managed cluster alternates between `offline` and `available` without any manual change to the environment or cluster. 

## Symptom: Cluster status changing from offline to available

When the network that connects the managed cluster to the hub cluster is unstable, the status of the managed cluster that is reported by the hub cluster cycles between `offline` and `available`. 

The connection between the hub cluster and managed cluster is maintained through a lease that is validated at the `leaseDurationSeconds` interval value. If the lease is not validated within five consecutive attempts of the `leaseDurationSeconds` value, then the cluster is marked `offline`. 

For example, the cluster is marked `offline` after five minutes with a `leaseDurationSeconds` interval of `60 seconds`. This configuration can be inadequate for reasons such as connectivity issues or latency, causing instability.

## Resolving the problem: Cluster status changing from offline to available

The five validation attempts is default and cannot be changed, but you can change the `leaseDurationSeconds` interval. 

Determine the amount of time, in minutes, that you want the cluster to be marked as `offline`, then multiply that value by 60 to convert to seconds. Then divide by the default five number of attempts. The result is your `leaseDurationSeconds` value.

1. Edit your `ManagedCluster` specification on the hub cluster by entering the following command, but replace `cluster-name` with the name of your managed cluster:

   ```
   oc edit managedcluster <cluster-name>
   ```
2. Increase the value of `leaseDurationSeconds` in your `ManagedCluster` specification, as seen in the following sample YAML:

   ```yaml
   apiVersion: cluster.open-cluster-management.io/v1
   kind: ManagedCluster
   metadata:
     name: <cluster-name>
   spec:
     hubAcceptsClient: true
     leaseDurationSeconds: 60
   ```
3. Save and apply the file.
