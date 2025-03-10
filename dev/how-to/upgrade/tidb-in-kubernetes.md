---
title: Perform a Rolling Update to a TiDB Cluster in Kubernetes
summary: Learn how to perform a rolling update to a TiDB cluster in Kubernetes.
category: how-to
---

# Perform a Rolling Update to a TiDB Cluster in Kubernetes

When you perform a rolling update to a TiDB cluster in Kubernetes, the Pod is shut down and recreated with the new image or/and configuration serially in the order of PD, TiKV, TiDB. Under the highly available deployment topology (minimum requirements: PD \* 3, TiKV \* 3, TiDB \* 2), performing a rolling update to PD and TiKV servers does not impact the running clients.

+ For the clients that can retry stale connections, performing a rolling update to TiDB servers neither impacts the running clients. 
+ For the clients that **can not** retry stale connections, performing a rolling update to TiDB servers will close the client connections and cause the request to fail. For this situation, it is recommended to add a function for the clients to retry, or to perform a rolling update to TiDB servers in idle time.

## Upgrade the version of TiDB cluster

1. Change the `image` of PD, TiKV and TiDB to different image versions in the `values.yaml` file.
2. Run the `helm upgrade` command:

    {{< copyable "shell-regular" >}}

    ```shell
    helm upgrade ${releaseName} pingcap/tidb-cluster -f values.yaml --version=v1.0.0-beta.3
    ```

3. Check the upgrade progress:

    {{< copyable "shell-regular" >}}

    ```shell
    watch kubectl -n ${namespace} get pod -o wide
    ```

## Change the configuration of TiDB cluster

By default, changes to the configuration files are applied to the TiDB cluster automatically through a rolling update. You can disable this feature by setting the `enableConfigMapRollout` variable to `false` in the `values.yaml` file, if so, the change of configuration will be loaded until the server being restarted.

You can change the configuration of TiDB cluster through the following steps:

1. Make sure the `enableConfigMapRollout` feature is not disabled explicitly in the `values.yaml` file.
2. Change the configurations in the `values.yaml` file as needed.
3. Run the `helm upgrade` command:

    {{< copyable "shell-regular" >}}

    ```shell
    helm upgrade ${releaseName} pingcap/tidb-cluster -f values.yaml --version=v1.0.0-beta.3
    ```

4. Check the upgrade process:

    {{< copyable "shell-regular" >}}

    ```shell
    watch kubectl -n ${namespace} get pod -o wide
    ```

> **Note:**
>
> - Changing the `enableConfigMapRollout` variable against a running cluster will trigger a rolling update of PD, TiKV, TiDB servers even if there's no change to the configuration.
> - Currently, PD's `scheduler` and `replication` configurations (the configuration key under `[scheduler]` and `[replication]` of the PD configuration file) can not be upgraded automatically.  You must upgrade them manually via `pd-ctl`. See [pd-ctl](/reference/tools/pd-control.md) for reference.
