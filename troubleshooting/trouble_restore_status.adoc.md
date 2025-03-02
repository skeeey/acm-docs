# Troubleshooting restore status finishes with errors

After you restore a backup, resources are restored correctly but the Red Hat Advanced Cluster Management restore resource shows a `FinishedWithErrors` status.

## Symptom: Troubleshooting restore status finishes with errors

Red Hat Advanced Cluster Management shows a `FinishedWithErrors` status and one or more of the Velero restore resources created by the Red Hat Advanced Cluster Management restore show a `PartiallyFailed` status.

## Resolving the problem: Troubleshooting restore status finishes with errors

If you restore from a backup that is empty, you can safely ignore the `FinishedWithErrors` status.

Red Hat Advanced Cluster Management for Kubernetes restore shows a cumulative status for all Velero restore resources. If one status is `PartiallyFailed` and the others are `Completed`, the cumulative status you see is `PartiallyFailed` to notify you that there is at least one issue.

To resolve the issue, check the status for all individual Velero restore resources with a `PartiallyFailed` status and view the logs for more details. You can get the log from the object storage directly, or download it from the OADP Operator by using the `DownloadRequest` custom resource.

To create a `DownloadRequest` from the console, complete the following steps:

1. Navigate to **Operators** > **Installed Operators** > **Create DownloadRequest**.
2. Select `BackupLog` as your **Kind** and follow the console instructions to complete the `DownloadRequest` creation.
