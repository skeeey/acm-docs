# Troubleshooting Submariner add-on status is degraded

After adding the Submariner add-on to the clusters in your cluster set, the status in the _Connection status_, _Agent status_, and _Gateway nodes_ show unexpected status for the clusters.

## Symptom: Submariner add-on status is degraded

You add the Submariner add-on to the clusters in your cluster set, the following status is shown in the _Gateway nodes_, _Agent status_, and _Connection status_ for the clusters:

* Gateway nodes labeled
  * `Progressing`: The process to label the gateway nodes started. 
  * `Nodes not labeled`: The gateway nodes are not labeled, possibly because the process to label them has not completed. 
  * `Nodes not labeled`: The gateway nodes are not yet labeled, possibly because the process is waiting for another process to finish.
  * Nodes labeled: The gateway nodes have been labeled.
* Agent status
  * Progressing: The installation of the Submariner agent started.
  * Degraded: The Submariner agent is not running correctly, possibly because it is still in progress.
* Connection status
  * Progressing: The process to establish a connection with the Submariner add-on started.
  * Degraded: The connection is not ready. If you just installed the add-on, the process might still be in progress. If it was after the connection has already been established and running, then two clusters have lost the connection to each other. When there are multiple clusters, all clusters display a `Degraded` status if any of the clusters is in adisconnected state.

It will also show which clusters are connected, and which ones are disconnected.

## Resolving the problem: Submariner add-on status is degraded

* The degraded status often resolves itself as the processes complete. You can see the current step of the process by clicking the status in the table. You can use that information to determine whether the process is finished and you need to take other troubleshooting steps.
* For an issue that does not resolve itself, complete the following steps to troubleshoot the problem: 
  1. You can use the `diagnose` command with the `subctl` utility to run some tests on the Submariner connections when the following conditions exist:

     1. The _Agent status_ or _Connection status_ is in a `Degraded` state. The `diagnose` command provides detailed analysis about the issue.
     2. Everything is green in console, but the networking connections are not working correctly. The `diagnose` command helps to confirm that there are no other connectivity or deployment issues outside of the console. It is considered best practice to run the `diagnostics` command after any deployment to identify issues.

        See [`diagnose`](https://submariner.io/operations/deployment/subctl/#diagnose) in the Submariner for more information about how to run the command. 
  2. If a problem continues with the `Connection status`, you can start by running the `diagnose` command of the `subctl` utility tool to get a more detailed status for the connection between two Submariner clusters. The format for the command is:

     ```
     subctl diagnose all --kubeconfig <path-to-kubeconfig-file>
     ```

     Replace `path-to-kubeconfig-file` with the path to the `kubeconfig` file. See [`diagnose`](https://submariner.io/operations/deployment/subctl/#diagnose) in the Submariner documentation for more information about the command.
  3. Check the firewall settings. Sometimes a problem with the connection is caused by firewall permissions issues that prevent the clusters from communicating. This can cause the `Connection status` to show as degraded. Run the following command to check the firewall issues:

     ```
     subctl diagnose firewall inter-cluster <path-to-local-kubeconfig> <path-to-remote-cluster-kubeconfig>
     ```

     Replace `path-to-local-kubeconfig` with the path to the `kubeconfig` file of one of the clusters.

     Replace `path-to-remote-kubeconfig` with the path to the `kubeconfig` file of the other cluster. you can run the `verify` command with your `subctl` utility tool to test the connection between two Submariner clusters. The basic format for the command is:
  4. If a problem continues with the `Connection status`, you can run the `verify` command with your `subctl` utility tool to test the connection between two Submariner clusters. The basic format for the command is:

     ```
     subctl verify --kubecontexts <cluster1>,<cluster2> [flags]
     ```

     Replace `cluster1` and `cluster2` with the names of the clusters that you are testing. See [`verify`](https://submariner.io/operations/deployment/subctl/#verify) in the Submariner documentation for more information about the command.
  5. After the troubleshooting steps resolve the issue, use the `benchmark` command with the `subctl` tool to establish a base on which to compare when you run additional diagnostics. 

     See [`benchmark`](https://submariner.io/operations/deployment/subctl/#benchmark) in the Submariner documentation for additional information about the options for the command.
