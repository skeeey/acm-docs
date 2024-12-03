# Troubleshooting installation status stuck in installing or pending

When installing Red Hat Advanced Cluster Management, the `MultiClusterHub` remains in `Installing` phase, or multiple pods maintain a `Pending` status.

## Symptom: Stuck in Pending status 

More than ten minutes passed since you installed `MultiClusterHub` and one or more components from the `status.components` field of the `MultiClusterHub` resource report `ProgressDeadlineExceeded`. Resource constraints on the cluster might be the issue. 

Check the pods in the namespace where `Multiclusterhub` was installed. You might see `Pending` with a status similar to the following:

```
reason: Unschedulable
message: '0/6 nodes are available: 3 Insufficient cpu, 3 node(s) had taint {node-role.kubernetes.io/master:
        }, that the pod didn't tolerate.'
```

In this case, the worker nodes resources are not sufficient in the cluster to run the product.

## Resolving the problem: Adjust worker node sizing

If you have this problem, then your cluster needs to be updated with either larger or more worker nodes. See [Sizing your cluster](../install/cluster_size.adoc#sizing-your-cluster) for guidelines on sizing your cluster.
