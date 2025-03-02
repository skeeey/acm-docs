# Troubleshooting Submariner not connecting after installation

If Submariner does not run correctly after you configure it, complete the following steps to diagnose the issue.

## Symptom: Submariner not connecting after installation

Your Submariner network is not communicating after installation.

## Identifying the problem: Submariner not connecting after installation

If the network connectivity is not established after deploying Submariner, begin the troubleshooting steps. Note that it might take several minutes for the processes to complete when you deploy Submariner.

## Resolving the problem: Submariner not connecting after installation

When Submariner does not run correctly after deployment, complete the following steps:

1. Check for the following requirements to determine whether the components of Submariner deployed correctly:

   * The `submariner-addon` pod is running in the `open-cluster-management` namespace of your hub cluster. 
   * The following pods are running in the `submariner-operator` namespace of each managed cluster:

     * submariner-addon
     * submariner-gateway
     * submariner-routeagent
     * submariner-operator
     * submariner-globalnet (only if Globalnet is enabled in the ClusterSet)
     * submariner-lighthouse-agent
     * submariner-lighthouse-coredns
     * submariner-networkplugin-syncer (only if the specified CNI value is `OVNKubernetes`)
     * submariner-metrics-proxy
2. Run the `subctl diagnose all` command to check the status of the required pods, with the exception of the `submariner-addon` pods. 
3. Make sure to run the `must-gather` command to collect logs that can help with debugging issues.
