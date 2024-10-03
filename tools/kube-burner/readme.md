# ArgoCD Performance Testing with Kube-Burner

This repository contains the configuration that is used to test ArgoCD's performance when managing multiple applications across different namespaces.

## Prerequisites

- **ArgoCD Installation**: Ensure that ArgoCD is installed and configured in your cluster.
- **OpenShift Cluster**: This setup is designed for an OpenShift cluster, but it should work on any Kubernetes-compatible environment.
- **Kube-Burner**: Install Kube-Burner for resource creation and metrics collection. You can find installation instructions on the [Kube-Burner GitHub page](https://github.com/kube-burner/kube-burner).

## Why Use Kube-Burner?

[Kube-Burner](https://kube-burner.github.io/kube-burner/latest/) is a versatile tool that creates resources, waits for them to reach a desired state, and collects relevant metrics. This makes it ideal for performance testing and benchmarking. In the context of ArgoCD, Kube-Burner can efficiently manage the lifecycle of ArgoCD applications by creating them based on predefined templates, waiting for their status to meet specific conditions (such as synchronization), and gathering key metrics. For example, the configuration below specifies the creation of ArgoCD Application resources, waits until they are in a "Synced" state using the customStatusPath for detailed status checks, and collects relevant metrics for performance analysis:

### Wait Configuration Example

The following example illustrates the wait configuration used in this setup:

```yaml
objects:
      - kind: Application
        objectTemplate: template/application.yaml
        replicas: 1
        waitOptions:
          forCondition: "Synced"
          customStatusPath: ".operationState.syncResult.resources[].status"
 ```

### Kube-Burner Configuration Details

#### Metrics Profile

The metric/metrics-profile.yaml file defines the metrics profile for ArgoCD performance testing, capturing essential metrics:

- ***CPU Usage of ArgoCD Application Controller:***
```
- query: irate(process_cpu_seconds_total{container="argocd-application-controller",namespace=~".+"}[1m])
  metricName: argocdAppControllerCPU
```
- ***Memory Usage of ArgoCD Application Controller:***
```
- query: go_memstats_heap_alloc_bytes{container="argocd-application-controller",namespace=~".+"}
  metricName: argocdAppControllerHeapAllocMemory

- query: go_memstats_heap_inuse_bytes{container="argocd-application-controller",namespace=~".+"}
  metricName: argocdAppControllerHeapInuseMemory
```
- ***Number of Kubernetes Resource Objects in the Cache:***
```
- query: sum(argocd_cluster_api_resource_objects{namespace=~".+"})
  metricName: argocdClusterApiResourceObjects
```  
- ***Number of Monitored Kubernetes API Resources:***
```
- query: sum(argocd_cluster_api_resources{namespace=~".+"})
  metricName: argocdClusterApiResources
```
- ***Total IO Operations of ArgoCD Application Controller:***
```
- query: sum(rate(container_fs_reads_total{pod=~"openshift-gitops-application-controller-.*",namespace=~".+"}[1m])) + sum(rate(container_fs_writes_total{pod=~"openshift-gitops-application-controller-.*",namespace=~".+"}[1m]))
  metricName: argocdAppControllerIO
```
- ***ArgoCD Pending Kubectl Exec:***
```
- query: sum(argocd_kubectl_exec_pending{namespace=~".+"})
  metricName: argocdPendingKubectlExec
```

### Sample Metrics Data

Below is a sample of metrics data collected using Kube-Burner:

```json
[
    {
        "timestamp": "2024-08-22T10:17:39.867Z",
        "labels": {
            "container": "argocd-application-controller",
            "endpoint": "metrics",
            "instance": "10.131.1.218:8082",
            "job": "openshift-gitops-metrics",
            "namespace": "openshift-gitops",
            "pod": "openshift-gitops-application-controller-0",
            "service": "openshift-gitops-metrics"
        },
        "value": 1281433176,
        "uuid": "1234",
        "query": "go_memstats_heap_alloc_bytes{container=\"argocd-application-controller\",namespace=~\".+\"}",
        "metricName": "argocdAppControllerHeapAllocMemory",
        "jobName": "create-applications"
    },
    {
        "timestamp": "2024-08-22T10:16:53.241Z",
        "labels": {
            "container": "argocd-application-controller",
            "endpoint": "metrics",
            "instance": "10.131.1.218:8082",
            "job": "openshift-gitops-metrics",
            "namespace": "openshift-gitops",
            "pod": "openshift-gitops-application-controller-0",
            "service": "openshift-gitops-metrics"
        },
        "value": 188994960,
        "uuid": "1234",
        "query": "go_memstats_heap_alloc_bytes{container=\"argocd-application-controller\",namespace=~\".+\"}",
        "metricName": "argocdAppControllerHeapAllocMemory",
        "jobName": "create-namespaces"
    },
    {
        "timestamp": "2024-08-22T10:17:04.303Z",
        "labels": {
            "container": "argocd-application-controller",
            "endpoint": "metrics",
            "instance": "10.131.1.218:8082",
            "job": "openshift-gitops-metrics",
            "namespace": "openshift-gitops",
            "pod": "openshift-gitops-application-controller-0",
            "service": "openshift-gitops-metrics"
        },
        "value": 202814368,
        "uuid": "1234",
        "query": "go_memstats_heap_alloc_bytes{container=\"argocd-application-controller\",namespace=~\".+\"}",
        "metricName": "argocdAppControllerHeapAllocMemory",
        "jobName": "create-role"
    }
]
```

### Metric Endpoint Configuration

To configure metric collection, update the metric/ep.yaml file as follows:
```yaml
endpoint: <metric-endpoint>
token: ""
metrics:
  - metric/metrics-profile.yaml # Path to the metrics query profile
indexer:
  type: local
  metricsDirectory: my-metrics # Directory where metrics will be collected
``` 
Replace <metric-endpoint> with the appropriate endpoint for your setup.

### Template Customization

Customize the YAML templates in the template directory before deployment:

- application.yaml: Update the namespace field with ```<argocd-namespace>```.

- namespace.yaml:
```yaml
labels:
  argocd.argoproj.io/managed-by: <argocd-namespace>
```  
- rolebinding.yaml:
```yaml
subjects:
  - kind: ServiceAccount
    name: <application-controller-service-account-name>
    namespace: <argocd-namespace>
```

Replace ```<argocd-namespace>``` and ```<application-controller-service-account-name>``` with the values relevant to your environment.

### Deploying the Test Setup
To deploy and start the performance test, navigate to the kube-burner directory and execute:

``` kube-burner init -c config.yaml -e metric/ep.yaml ```
