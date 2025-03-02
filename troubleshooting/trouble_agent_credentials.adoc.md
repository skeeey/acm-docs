# Troubleshooting rotating credentials for agents

Agents are responsible for connections. See how you can rotate the following credentials:

* `registration-agent`: Connects the registration agent to the hub cluster.
* `work-agent`: Connects the work agent to the hub cluster.
 
## Symptom: Expired credentials for agents

When your credentials expire, you can manually rotate them.

## Resolving the problem: Manually rotate credentials for agents

To rotate credentials, delete the `hub-kubeconfig` secret to restart the registration pods.
 
 - `APIServer`: Connects agents and add-ons to the hub cluster.
 
+
. On the hub cluster, display the import command by entering the following command: 

+
```
oc get secret -n ${CLUSTER_NAME} ${CLUSTER_NAME}-import -ojsonpath='{.data.import\.yaml}' | base64 --decode  > import.yaml
```

+
. On the managed cluster, apply the `import.yaml` file. Run the following command: `oc apply -f import.yaml`.
