---
title: Tutorials
---

By default, the Helm Broker fetches bundles listed in the `index.yaml` file from the `bundles` repository [release](https://github.com/kyma-project/bundles/releases). This tutorial shows how to configure the Helm Broker to fetch bundle definitions from other remote HTTPS servers.

## Steps

1. [Create a repository](#details-create-a-bundles-repository) with your bundles. To complete this tutorial step by step, use the existing [bundles](https://github.com/kyma-project/bundles/tree/master/bundles) repository.
2. [Install Kyma](/root/kyma/#installation-installation) locally or on a cluster.
3. Create a ConfigMap which contains an URL to the repository. You can either:

  * Create a ConfigMap using the `kubectl create` command:

    ```bash
    kubectl create configmap my-helm-repos-urls -n kyma-system --from-literal=URLs=https://github.com/kyma-project/bundles/releases/download/0.3.0/index-testing.yaml
    ```
    >**NOTE:** If you want to fetch bundles from many HTTP servers, use `\n` to separate the URLs.

    If you use this method, you must label your ConfigMap with `helm-broker-repo=true`. To add the label to your ConfigMap, run:
    ```bash
    kubectl label configmap my-helm-repos-urls -n kyma-system helm-broker-repo=true
    ```

  * Create a ConfigMap from the following example:

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: my-helm-repos-urls
      labels:
        helm-broker-repo: "true"
        data:
          URLs: |-
            https://github.com/kyma-project/bundles/releases/download/0.3.0/index-testing.yaml
    ```

    Save your ConfigMap as `my-helm-repos-urls.yaml`. Then, run:
    ```bash
    kubectl apply -f my-helm-repos-urls.yaml
    ```
    >**NOTE:** Your bundle repository must contain at least one file named `index.yaml` as the Helm Broker automatically searches for it when you provide the `https://{host}/{path}/{bundle_repo_version}/` URL to your ConfigMap.

  The default ConfigMap provided by the Helm Broker is the [`helm-repos-urls`](https://github.com/kyma-project/kyma/blob/master/resources/helm-broker/templates/cfg-repos-url.yaml) ConfigMap. Do not edit this ConfigMap. Create a separate one instead. Depending on your needs and preferences, you can create one or more ConfigMaps with URLs to different remote HTTPS servers.

4. The Helm Broker triggers the Service Catalog synchronization automatically. New Service Classes appear after a few seconds.

## Troubleshooting

These are the rules you must follow when you configure the Helm Broker to fetch bundle definitions from remote HTTPS servers. Otherwise, the Helm Broker will not display your bundles.
* If you use your bundle in two different repositories simultaneously, the Helm Broker detects a conflict and does not display this bundle at all. You can see the details of the conflict in the Helm Broker application logs. If you need a given bundle in two or more repositories, do not use them at the same time.
* On your non-local clusters, you can use only servers with TLS enabled. All incorrect or unsecured URLs will be omitted. Find the information about the rejected URLs in the Helm Broker logs. You can use unsecured URLs only on your local cluster. To use URLs without TLS enabled, set the **global.isDevelopMode** environment variable in the [values.yaml](https://github.com/kyma-project/kyma/blob/master/resources/helm-broker/values.yaml) file to `true`.
