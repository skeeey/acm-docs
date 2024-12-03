# Multicluster global hub Operator known issues

Review the known issues for the multicluster global hub Operator. The following list contains known issues for this release, or known issues that continued from the previous release. For your OpenShift Container Platform cluster, see [OpenShift Container Platform known issues](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/release_notes/#ocp-4-15-known-issues).

## The detached managed hub cluster deletes and recreates the namespace and resources 

If you import a managed hub cluster in the hosted mode and detach this managed hub cluster, then it deletes and recreates the `open-cluster-management-agent-addon` namespace. The detached managed hub cluster also deletes and recreates all the related `addon` resources within this namespace. 

There is currently no workaround for this issue. 

## The multicluster global hub Search is not enabled 

If you installed your multicluster engine operator into a customized namespace, then you cannot enable the multicluster global hub Search. In this case, the search operator fails to update the `console-mce-config` config map because the `globalSearchFeatureFlag=true` flag is added, preventing you from enabling the multicluster global hub Search.

To workaround this issue, you must manually place the `console-mce-config` config map in the correct namespace by running the following command: 

```bash
oc patch configmap console-mce-config -n {{MCE_NAMESPACE}} -p '{"data": {"globalSearchFeatureFlag": "enabled"}}'
```

## Kafka operator keeps restarting 

In the Federal Information Processing Standard (FIPS) environment, the Kafka operator keeps restarting because of the out-of-memory (OOM) state. To fix this issue, set the resource limit to at least `512M`. For detailed steps on how to set this limit, see [amq stream doc](https://docs.redhat.com/documentation/en-us/red_hat_amq_streams/2.6/html/deploying_and_managing_amq_streams_on_openshift/deploy-intro_str#assembly-fips-support-str).

## Backup and restore known issues 

If your original multicluster global hub cluster crashes, the multicluster global hub loses its generated events and `cron` jobs. Even if you restore the new multicluster global hub cluster, the events and `cron` jobs are not restored. To workaround this issue, you can manually run the `cron` job, see [Running the summarization process manually](https://docs.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.9/html/multicluster_global_hub/multicluster-global-hub#global-hub-compliance-manual).

## Managed cluster displays but is not counted

A managed cluster that is not created successfully, meaning `clusterclaim id.k8s.io` does not exist in the managed cluster, is not counted in the policy compliance dashboards, but shows in the policy console. 

## The multicluster global hub is installed on OpenShift Container Platform 4.13 hyperlinks might redirect home

If the multicluster global hub Operator is installed on OpenShift Container Platform 4.13, all hyperlinks that link to the managed clusters list and detail pages in dashboards might redirect to the Red Hat Advanced Cluster Management home page. 

You need to manually go to your target page.

## The standard group filter cannot pass to the new page

In the _Global Hub Policy Group Compliancy Overview_ hub dashboards, you can check one data point by clicking **View Offending Policies for standard group**, but after you click this link to go to the offending page, the standard group filter cannot pass to the new page. 

This is also an issue for the **Cluster Group Compliancy Overview**.
