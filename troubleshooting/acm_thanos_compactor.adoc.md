# Troubleshooting a block error for Thanos compactor

You might receive a block error message that indicates that the block for Thanos compactor is corrupted.

## Symptom: Block error for Thanos compactor

After you upgrade Red Hat Advanced Cluster Management for Kubernetes and check the logs for the Thanos compactor by using the `oc logs observability-thanos-compact-0` command, the logs display the following error message:

```
ts=2024-01-24T15:34:51.948653839Z caller=compact.go:491 level=error msg="critical error detected; halting" err="compaction: group 0@15699422364132557315: compact blocks [/var/thanos/compact/compact/0@15699422364132557315/01HKZGQGJCKQWF3XMA8EXAMPLE /var/thanos/compact/compact/0@15699422364132557315/01HKZQK7TD06J2XWGR5EXAMPLE /var/thanos/compact/compact/0@15699422364132557315/01HKZYEZ2DVDQXF1STVEXAMPLE /var/thanos/compact/compact/0@15699422364132557315/01HM05APAHXBQSNC0N5EXAMPLE]: populate block: chunk iter: cannot populate chunk 8 from block 01HKZYEZ2DVDQXF1STVEXAMPLE: segment index 0 out of range"
```

## Resolving the problem: Add the _thanos bucket verify_ command

Add the `thanos bucket verify` command to the object storage configuration. Complete the following steps:

1. Resolve the block error by adding the `thanos bucket verify` command to the object storage configuration. Set the configuration in the `observability-thanos-compact` pod by using the following commands:

+
```bash
oc rsh observability-thanos-compact-0
[..]
thanos tools bucket verify -r --objstore.config="$OBJSTORE_CONFIG" --objstore-backup.config="$OBJSTORE_CONFIG" --id=01HKZYEZ2DVDQXF1STVEXAMPLE
```

1. If the previous command does not work, you must mark the block for deletion because it might be corrupted. Run the following commands:

+
```bash
thanos tools bucket mark --id "01HKZYEZ2DVDQXF1STVEXAMPLE" --objstore.config="$OBJSTORE_CONFIG" --marker=deletion-mark.json --details=DELETE 
```

1. If you blocked for deletion, clean up the marked blocks by running the following command:

+
```bash
thanos tools bucket cleanup --objstore.config="$OBJSTORE_CONFIG"
```
