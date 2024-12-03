# Troubleshooting Submariner end-to-end test failures 

After running Submariner end-to-end tests, you might get failures. Use the following sections to help you troubleshoot these end-to-end test failures.  

## Symptom: Submariner end-to-end data plane test fails

When the end-to-end data plane test fails, the Submariner tests show that the `connector` pod can connect to the `listener` pod, but later the `connector` pod gets stuck in the `listening` phase. 

## Resolving the problem: Submariner end-to-end data plane test fails

The maximum transmission unit (MTU) can cause the end-to-end data plane test failure. For example, the MTU might cause the `inter-cluster` traffic over the Internet Protocol Security (IPsec) to fail. Verify if the MTU causes the failure by running an end-to-end data plane test that uses a small packet size. 

To run this type of test, run the following command in your Submariner workspace:

```bash
subctl verify --verbose --only connectivity --context <from_context> --tocontext <to_context> --image-override submariner-nettest=quay.io/submariner/nettest:devel --packet-size 200
```

If the test succeeds with this small packet size, you can resolve the connection issues by setting the transmission control protocol (TCP) maximum segment size (MSS). Set the TCP MSS by completing the following steps: 

1. Set the TCP MSS `clamping` value by annotating the gateway node. For example, run the following command with a value of `1200`: 

+
```bash
oc annotate node <node_name> submariner.io/tcp-clamp-mss=1200
```

1. Restart all the `RouteAgent` pods by running the following command: 

+
```bash
oc delete pod -n submariner-operator -l app=submariner-routeagent 
```

## Symptom: Submariner end-to-end test fails for bare-metal clusters 

The end-to-end data plane tests might fail for the bare-metal cluster if the container network interface (CNI) is OpenShiftSDN, or if the virtual extensible local-area network (VXLAN) is used for the `inter-cluster` tunnels.

## Resolving the problem: Submariner end-to-end test fails for bare-metal clusters 

A bug in the User Datagram Protocal (UDP) checksum calculation by the hardware can be the root cause for the end-to-end data plane test failures for bare-metal clusters. To troubleshoot this bug, disable the hardware offloading by applying the following YAML file:  

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata: 
  name: disable-offload
  namespace: submariner-operator
spec: 
  selector: 
    matchLabels: 
      app: disable-offload
  template: 
    metadata: 
      labels: 
        app: disable-offload
    spec: 
      tolerations: 
      - operator: Exists  
      containers: 
        - name: disable-offload
          image: nicolaka/netshoot
          imagePullPolicy: IfNotPresent
          securityContext: 
            allowPrivilegeEscalation: true
            capabilities: 
              add: 
              - net_admin
              drop: 
              - all
            privileged: true
            readOnlyRootFilesystem: false
            runAsNonRoot: false
          command: ["/bin/sh", "-c"]
          args: 
            - ethtool --offload vxlan-tunnel rx off tx off;
              ethtool --offload vx-submariner rx off tx off;
      restartPolicy: Always
      securityContext: {}
      serviceAccount: submariner-routeagent
      serviceAccountName: submariner-routeagent
      hostNetwork: true 
```
