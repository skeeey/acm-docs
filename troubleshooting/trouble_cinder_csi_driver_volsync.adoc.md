# Troubleshooting the cinder Container Storage Interface (CSI) driver for VolSync

If you use VolSync or use a default setting in a cinder Container Storage Interface (CSI) driver, you might encounter errors for the PVC that is in use. 

## Symptom: `Volumesnapshot` error state

You can configure a VolSync `ReplicationSource` or `ReplicationDestination` to use snapshots. Also, you can configure the `storageclass` and `volumesnapshotclass` in the `ReplicationSource` and `ReplicationDestination`. There is a parameter on the cinder  `volumesnapshotclass` called `force-create` with a default value of `false`. This `force-create` parameter on the `volumesnapshotclass` means cinder does not allow the `volumesnapshot` to be taken of a PVC in use. As a result, the `volumesnapshot` is in an error state. 

## Resolving the problem: Setting the parameter to `true`

1. Create a new `volumesnapshotclass` for the cinder CSI driver.
2. Change the paramater, `force-create`, to `true`. See the following sample YAML:

+
```yaml
apiVersion: snapshot.storage.k8s.io/v1
deletionPolicy: Delete
driver: cinder.csi.openstack.org
kind: VolumeSnapshotClass
metadata:
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: 'true'
  name: standard-csi
parameters:
  force-create: 'true'
```
