# Managed cluster creation fails on Red Hat OpenStack Platform with unknown authority error

If you experience a problem when creating a Red Hat OpenShift Container Platform cluster on Red Hat OpenStack Platform, see the following troubleshooting information to see if one of them addresses your problem.

## Symptom: Managed cluster creation fails with unknown authority error

After creating a new Red Hat OpenShift Container Platform cluster on Red Hat OpenStack Platform using self-signed certificates, the cluster fails with an error message that indicates an unknown authority error.

## Identifying the problem: Managed cluster creation fails with unknown authority error

The deployment of the managed cluster fails and returns the following error message:

`x509: certificate signed by unknown authority`

## Resolving the problem: Managed cluster creation fails with unknown authority error

Verify that the following files are configured correctly:

1. The `clouds.yaml` file must specify the path to the `ca.crt` file in the `cacert` parameter. The `cacert` parameter is passed to the OpenShift installer when generating the ignition shim. See the following example:

   ```yaml
   clouds:
     openstack:
       cacert: "/etc/pki/ca-trust/source/anchors/ca.crt"
   ```
2. The `certificatesSecretRef` paremeter must reference a secret with a file name matching the `ca.crt` file. See the following example:

   ```yaml
   spec:
     baseDomain: dev09.red-chesterfield.com
     clusterName: txue-osspoke
     platform:
       openstack:
         cloud: openstack
         credentialsSecretRef:
           name: txue-osspoke-openstack-creds
         certificatesSecretRef:
           name: txue-osspoke-openstack-certificatebundle
   ```

   To create a secret with a matching file name, run the following command:

   ```
   oc create secret generic txue-osspoke-openstack-certificatebundle --from-file=ca.crt=ca.crt.pem -n $CLUSTERNAME
   ```

3. The size of the `ca.cert` file must be less than 63 thousand bytes.
