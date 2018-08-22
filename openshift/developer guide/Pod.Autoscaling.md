# Pod Autoscaling

## Overview(概述)

A horizontal pod autoscaler, defined by a HorizontalPodAutoscaler object, specifies how the system should automatically increase or decrease the scale of a replication controller or deployment configuration, based on metrics collected from the pods that belong to that replication controller or deployment configuration.

```text
由HorizontalPodAutoscaler对象定义的水平pod自动调节器指定系统应如何根据从属于该复制控制器或部署配置的pod收集的度量标准自动增加或减少复制控制器或部署配置的规模。
```

## Requirements for Using Horizontal Pod Autoscalers(使用Horizontal Pod Autoscalers的要求)

In order to use horizontal pod autoscalers, your cluster administrator must have properly "configured cluster metrics".

```text
要使用水平pod自动跟踪器，群集管理员必须具有正确配置的群集指标。
```

## Supported Metrics(支持的指标)

The following metrics are supported by horizontal pod autoscalers:

Table 1. Metrics
Metric|Description|API version
-| :-: | -:
CPU utilization|Percentage of the requested CPU|autoscaling/v1, autoscaling/v2beta1
Memory utilization | Percentage of the requested memory. | autoscaling/v2beta1

## Autoscaling(自动扩展)

You can create a horizontal pod autoscaler with the oc autoscale command and specify the minimum and maximum number of pods you want to run, as well as the CPU utilization or memory utilization your pods should target.

```text
您可以使用oc autoscale命令创建水平pod自动缩放器，并指定要运行的pod的最小和最大数量，以及pod应定位的CPU利用率或内存利用率。
```

Autoscaling for Memory Utilization is a Technology Preview feature only.

```text
内存利用率自动缩放仅是技术预览功能。
```

After a horizontal pod autoscaler is created, it begins attempting to query Heapster for metrics on the pods. It may take one to two minutes before Heapster obtains the initial metrics.

```text
创建水平pod自动缩放器后，它会开始尝试向Heapster查询pod上的指标。 在Heapster获得初始指标之前可能需要一到两分钟。
```

After metrics are available in Heapster, the horizontal pod autoscaler computes the ratio of the current metric utilization with the desired metric utilization, and scales up or down accordingly. The scaling will occur at a regular interval, but it may take one to two minutes before metrics make their way into Heapster.

```text
在Heapster中提供度量标准后，Horizontal Pod Autoscalers将计算当前度量标准利用率与所需度量标准利用率的比率，并相应地向上或向下扩展。 缩放将定期发生，但在指标进入Heapster之前可能需要一到两分钟。
```

For replication controllers, this scaling corresponds directly to the replicas of the replication controller. For deployment configurations, scaling corresponds directly to the replica count of the deployment configuration. Note that autoscaling applies only to the latest deployment in the Complete phase.

```text
对于复制控制器，此扩展直接对应于复制控制器的副本。 对于部署配置，扩展直接对应于部署配置的副本计数。 请注意，自动缩放仅适用于“完成”阶段中的最新部署。
```

OKD automatically accounts for resources and prevents unnecessary autoscaling during resource spikes, such as during start up. Pods in the unready state have 0 CPU usage when scaling up and the autoscaler ignores the pods when scaling down. Pods without known metrics have 0% CPU usage when scaling up and 100% CPU when scaling down. This allows for more stability during the HPA decision. To use this feature, you must configure readiness checks to determine if a new pod is ready for use.

```text
OKD会自动对资源进行核算，并防止在资源高峰期间（例如启动期间）进行不必要的自动扩展。 处于未准备状态的窗格在向上扩展时具有0 CPU使用率，并且自动缩放器在缩小时忽略窗格。 没有已知指标的窗格在扩展时具有0％的CPU使用率，在缩小时具有100％的CPU。 这样可以在HPA决策期间获得更高的稳定性。 要使用此功能，您必须配置准备情况检查以确定是否可以使用新容器。
```

## Autoscaling for CPU Utilization(CPU使用率自动扩展)

Use the oc autoscale command and specify at least the maximum number of pods you want to run at any given time. You can optionally specify the minimum number of pods and the average CPU utilization your pods should target, otherwise those are given default values from the OKD server.

```text
使用oc autoscale命令并至少指定要在任何给定时间运行的最大pod数。 您可以选择指定pod的最小数量以及pod应该定位的平均CPU利用率，否则这些将从OKD服务器获得默认值。
```

For example:

```shell
$ oc autoscale dc/frontend --min 1 --max 10 --cpu-percent=80
deploymentconfig "frontend" autoscaled
```

The above example creates a horizontal pod autoscaler with the following definition when using the autoscaling/v1 version of the horizontal pod autoscaler:

```text
在使用水平pod autoscaler的autoscaling/v1版本时，上面的示例创建了一个具有以下定义的水平pod自动缩放器：
```

    apiVersion: autoscaling/v1
    kind: HorizontalPodAutoscaler
    metadata:
        name: frontend (1)
    spec:
        scaleTargetRef:
            kind: DeploymentConfig (2)
            name: frontend (3)
            apiVersion: apps/v1 (4)
            subresource: scale
        minReplicas: 1 (5)
        maxReplicas: 10 (6)
        cpuUtilization:
            targetCPUUtilizationPercentage: 80 (7)

(1)The name of this horizontal pod autoscaler object
(2)The kind of object to scale
(3)The name of the object to scale
(4)The API version of the object to scale
(5)The minimum number of replicas to which to scale down
(6)The maximum number of replicas to which to scale up
(7)The percentage of the requested CPU that each pod should ideally be using

```text
（1）此水平pod自动缩放器对象的名称
（2）要扩展的对象类型
（3）要缩放的对象的名称
（4）要扩展的对象的API版本
（5）缩小的最小副本数量
（6）要扩展的最大副本数
（7）理想情况下每个pod应使用的请求CPU的百分比
```

Alternatively, the oc autoscale command creates a horizontal pod autoscaler with the following definition when using the v2beta1 version of the horizontal pod autoscaler:

```text
或者，当使用v2beta1版本的水平pod自动缩放器时，oc autoscale命令会创建一个具有以下定义的水平pod自动缩放器：
```

    apiVersion: autoscaling/v2beta1
    kind: HorizontalPodAutoscaler
    metadata:
    name: hpa-resource-metrics-cpu (1)
    spec:
    scaleTargetRef:
        apiVersion: apps/v1 (2)
        kind: ReplicationController (3)
        name: hello-hpa-cpu (4)
    minReplicas: 1 (5)
    maxReplicas: 10 (6)
    metrics:
    - type: Resource
        resource:
        name: cpu
        targetAverageUtilization: 50 (7)

(1)The name of this horizontal pod autoscaler object
(2)The API version of the object to scale
(3)The kind of object to scale
(4)The name of the object to scale
(5)The minimum number of replicas to which to scale down
(6)The maximum number of replicas to which to scale up
(7)The average percentage of the requested CPU that each pod should be using

```text
（1）此水平pod自动缩放器对象的名称
（2）要扩展的对象的API版本
（3）要扩展的对象类型
（4）要缩放的对象的名称
（5）缩小的最小副本数量
（6）要扩展的最大副本数
（7）每个pod应使用的请求CPU的平均百分比
```

## Autoscaling for Memory Utilization(内存使用率自动扩展)

Autoscaling for Memory Utilization is a Technology Preview feature only.

```text
内存利用率自动缩放仅是技术预览功能。
```

Unlike CPU-based autoscaling, memory-based autoscaling requires specifying the autoscaler using YAML instead of using the oc autoscale command. Optionally, you can specify the minimum number of pods and the average memory utilization your pods should target as well, otherwise those are given default values from the OKD server.

```text
与基于CPU的自动缩放不同，基于内存的自动缩放需要使用YAML指定自动缩放器，而不是使用oc autoscale命令。 （可选）您可以指定pod的最小数量以及pod应该定位的平均内存利用率，否则这些将从OKD服务器获得默认值。
```

Memory-based autoscaling is only available with the v2beta1 version of the autoscaling API. Enable memory-based autoscaling by adding the following to your cluster’s master-config.yaml file:

```text
基于内存的自动扩展仅适用于v2beta1版本的自动扩展API。 通过将以下内容添加到群集的master-config.yaml文件中，启用基于内存的自动缩放：
```

    ...
    apiServerArguments:
    runtime-config:
    - apis/autoscaling/v2beta1=true
    ...

Place the following in a file, such as hpa.yaml:

    apiVersion: autoscaling/v2beta1
    kind: HorizontalPodAutoscaler
    metadata:
    name: hpa-resource-metrics-memory (1)
    spec:
    scaleTargetRef:
        apiVersion: apps/v1 (2)
        kind: ReplicationController (3)
        name: hello-hpa-memory (4)
    minReplicas: 1 (5)
    maxReplicas: 10 (6)
    metrics:
    - type: Resource
        resource:
        name: memory
        targetAverageUtilization: 50 (7)

(1)The name of this horizontal pod autoscaler object
(2)The API version of the object to scale
(3)The kind of object to scale
(4)The name of the object to scale
(5)The minimum number of replicas to which to scale down
(6)The maximum number of replicas to which to scale up
(7)The average percentage of the requested memory that each pod should be using

```text
（1）此水平pod自动缩放器对象的名称
（2）要扩展的对象的API版本
（3）要扩展的对象类型
（4）要缩放的对象的名称
（5）缩小的最小副本数量
（6）要扩展的最大副本数
（7）每个pod应使用的请求内存的平均百分比
```

Then, create the autoscaler from the above file:

```shell
oc create -f hpa.yaml
```

For memory-based autoscaling to work, memory usage must increase and decrease proportionally to the replica count. On average:

```text
要使基于内存的自动缩放工作，内存使用量必须与副本计数成比例地增加和减少。 一般：
```

* An increase in replica count must lead to an overall decrease in memory (working set) usage per-pod.

* A decrease in replica count must lead to an overall increase in per-pod memory usage.

```text
副本计数的增加必然导致每个pod的内存（工作集）使用率整体下降。

副本计数的减少必然导致每个pod的内存使用量整体增加。
```

Use the OpenShift web console to check the memory behavior of your application and ensure that your application meets these requirements before using memory-based autoscaling.

```text
使用OpenShift Web控制台检查应用程序的内存行为，并确保在使用基于内存的自动缩放之前，您的应用程序满足这些要求。
```

## Viewing a Horizontal Pod Autoscaler

Use the oc get command to view information on the CPU utilization and pod limits:

```text
使用oc get命令查看有关CPU利用率和pod限制的信息：
```

$ oc get hpa/hpa-resource-metrics-cpu
NAME                         REFERENCE                                 TARGET    CURRENT  MINPODS        MAXPODS    AGE
hpa-resource-metrics-cpu     DeploymentConfig/default/frontend/scale   80%       79%      1              10         8d

The output includes the following:

* Target. The targeted average CPU utilization across all pods controlled by the deployment configuration.

* Current. The current CPU utilization across all pods controlled by the deployment configuration.

* Minpods/Maxpods. The minimum and maximum number of replicas that can be set by the autoscaler.

```text
目标。 由部署配置控制的所有pod的目标平均CPU利用率。

当前。 由部署配置控制的所有pod的当前CPU利用率。

Minpods/ Maxpods。 自动缩放器可以设置的最小和最大副本数。
```

Use the oc describe command for detailed information on the horizontal pod autoscaler object.

```text
有关水平pod autoscaler对象的详细信息，请使用oc describe命令。
```

    $ oc describe hpa/hpa-resource-metrics-cpu
    Name:                           hpa-resource-metrics-cpu
    Namespace:                      default
    Labels:                         <none>
    CreationTimestamp:              Mon, 26 Oct 2015 21:13:47 -0400
    Reference:                      DeploymentConfig/default/frontend/scale
    Target CPU utilization:         80% (1)
    Current CPU utilization:        79% (2)
    Min replicas:                   1 (3)
    Max replicas:                   4 (4)
    ReplicationController pods:     1 current / 1 desired
    Conditions: (5)
    Type                  Status  Reason                  Message
    ----                  ------  ------                  -------
    AbleToScale           True    ReadyForNewScale        the last scale time was sufficiently old as to warrant a new scale
    ScalingActive         True    ValidMetricFound        the HPA was able to successfully calculate a replica count from pods metric http_requests
    ScalingLimited        False   DesiredWithinRange      the desired replica count is within the acceptable range
    Events:

(1)The average percentage of the requested memory that each pod should be using.
(2)The current CPU utilization across all pods controlled by the deployment configuration.
(3)The minimum number of replicas to scale down to.
(4)The maximum number of replicas to scale up to.
(5)If the object used the v2alpha1 API, status conditions are displayed.

```text
（1）每个pod应使用的请求内存的平均百分比。
（2）部署配置控制的所有pod的当前CPU利用率。
（3）缩小到的最小副本数量。
（4）要扩展到的最大副本数。
（5）如果对象使用v2alpha1 API，则显示状态条件。
```

## Viewing Horizontal Pod Autoscaler Status Conditions(查看水平Pod自动缩放器状态条件)

You can use the status conditions set to determine whether or not the horizontal pod autoscaler is able to scale and whether or not it is currently restricted in any way.

```text
您可以使用设置的状态条件来确定水平pod自动缩放器是否能够扩展以及当前是否以任何方式限制它。
```

The horizontal pod autoscaler status conditions are available with the v2beta1 version of the autoscaling API:

```text
v2beta1版本的自动缩放API提供水平pod自动调节器状态条件：
```

    kubernetesMasterConfig:
        ...
        apiServerArguments:
            runtime-config:
            - apis/autoscaling/v2beta1=true

The following status conditions are set:

```text
设置以下状态条件：
```

AbleToScale indicates whether the horizontal pod autoscaler is able to fetch and update scales, and whether any backoff conditions are preventing scaling.

```text
AbleToScale指示水平pod自动调节器是否能够获取和更新scales，以及是否有任何后退条件阻止缩放。
```

* A True condition indicates scaling is allowed.

* A False condition indicates scaling is not allowed for the reason specified.

```text
True条件表示允许缩放。

False条件表示由于指定的原因不允许缩放。
```

ScalingActive indicates whether the horizontal pod autoscaler is enabled (the replica count of the target is not zero) and is able to calculate desired scales.

```text
ScalingActive指示是否启用水平pod自动缩放器（目标的副本计数不为零）并且能够计算所需的比例。
```

* A True condition indicates metrics is working properly.

* A False condition generally indicates a problem with fetching metrics.

```text
True条件表示指标正常运行。

False条件通常表示获取指标存在问题。
```

ScalingLimited indicates that autoscaling is not allowed because a maximum or minimum replica count was reached.

```text
ScalingLimited表示不允许自动缩放，因为达到了最大或最小副本计数。
```

* A True condition indicates that you need to raise or lower the minimum or maximum replica count in order to scale.

* A False condition indicates that the requested scaling is allowed.

```text
True条件表示您需要提高或降低最小或最大副本计数才能进行缩放。

False条件表示允许所请求的缩放。
```

If you need to add or edit this line, restart the OKD services:

    # systemctl restart origin-master-api origin-master-controllers

To see the conditions affecting a horizontal pod autoscaler, use oc describe hpa. Conditions appear in the status.conditions field:

```text
要查看影响水平pod自动缩放器的条件，请使用oc describe hpa。 条件显示在status.conditions字段中：
```

    $ oc describe hpa cm-test
    Name:                           cm-test
    Namespace:                      prom
    Labels:                         <none>
    Annotations:                    <none>
    CreationTimestamp:              Fri, 16 Jun 2017 18:09:22 +0000
    Reference:                      ReplicationController/cm-test
    Metrics:                        ( current / target )
        "http_requests" on pods:      66m / 500m
    Min replicas:                   1
    Max replicas:                   4
    ReplicationController pods:     1 current / 1 desired
    Conditions: (1)
        Type                  Status  Reason                  Message
        ----                  ------  ------                  -------
        AbleToScale       True      ReadyForNewScale    the last scale time was sufficiently old as to warrant a new scale
        ScalingActive     True      ValidMetricFound    the HPA was able to successfully calculate a replica count from pods metric http_request
        ScalingLimited    False     DesiredWithinRange  the desired replica count is within the acceptable range
    Events:

(1)The horizontal pod autoscaler status messages.

* The AbleToScale condition indicates whether HPA is able to fetch and update scales, as well as whether any backoff-related conditions would prevent scaling.

```text
AbleToScale条件指示HPA是否能够获取和更新比例，以及任何与退避相关的条件是否会阻止扩展。
```

* The ScalingActive condition indicates whether the HPA is enabled (for example, the replica count of the target is not zero) and is able to calculate desired scales. A`False` status generally indicates problems with fetching metrics.

```text
ScalingActive条件指示HPA是否已启用（例如，目标的副本计数不为零）并且能够计算所需的比例。 A`False`状态通常表示获取指标的问题。
```

* The ScalingLimited condition indicates that the desired scale was capped by the maximum or minimum of the horizontal pod autoscaler. A`False` status generally indicates that you might need to raise or lower the minimum or maximum replica count constraints on your horizontal pod autoscaler.

```text
ScalingLimited条件表示所需的比例由水平pod自动缩放器的最大值或最小值限制。 A`False`状态通常表示您可能需要提高或降低水平pod自动缩放器上的最小或最大副本计数约束。
```

The following is an example of a pod that is unable to scale:

    Conditions:
        Type           Status    Reason            Message
        ----           ------    ------            -------
        AbleToScale    False     FailedGetScale    the HPA controller was unable to get the target's current scale: replicationcontrollers/scale.extensions "hello-hpa-cpu" not found

The following is an example of a pod that could not obtain the needed metrics for scaling:

    Conditions:
        Type                  Status    Reason                    Message
        ----                  ------    ------                    -------
        AbleToScale           True     SucceededGetScale          the HPA controller was able to get the target's current scale
        ScalingActive         False    FailedGetResourceMetric    the HPA was unable to compute the replica count: unable to get metrics for resource cpu: no metrics returned from heapster

The following is an example of a pod where the requested autoscaling was less than the required minimums:

    Conditions:
        Type              Status    Reason              Message
        ----              ------    ------              -------
        AbleToScale       True      ReadyForNewScale    the last scale time was sufficiently old as to warrant a new scale
        ScalingActive     True      ValidMetricFound    the HPA was able to successfully calculate a replica count from pods metric http_request
        ScalingLimited    False     DesiredWithinRange  the desired replica count is within the acceptable range
    Events: