# Using Daemonsets

## Overview

A daemonset can be used to run replicas of a pod on specific or all nodes in an OKD cluster.

```text
守护进程可用于在OKD集群中的特定或所有节点上运行pod的副本。
```

Use daemonsets to create shared storage, run a logging pod on every node in your cluster, or deploy a monitoring agent on every node.

```text
使用daemonsets创建共享存储，在群集中的每个节点上运行日志记录窗格，或在每个节点上部署监视代理程序。
```

For security reasons, only cluster administrators can create daemonsets. (Granting Users Daemonset Permissions.)

```text
出于安全原因，只有集群管理员才能创建守护程序集。 （授予用户Daemonset权限。）
```

For more information on daemonsets, see the Kubernetes documentation.

```text
有关守护程序集的更多信息，请参阅Kubernetes文档。
```

Daemonset scheduling is incompatible with project’s default node selector. If you fail to disable it, the daemonset gets restricted by merging with the default node selector. This results in frequent pod recreates on the nodes that got unselected by the merged node selector, which in turn puts unwanted load on the cluster.

```text
守护进程调度与项目的默认节点选择器不兼容。 如果未能将其禁用，则通过与默认节点选择器合并来限制守护进程。 这会导致在合并节点选择器未选中的节点上频繁重新创建pod，从而在集群上放置不必要的负载。
```

Therefore,

Before you start using daemonsets, disable the default project-wide node selector in your namespace, by setting the namespace annotation openshift.io/node-selector to an empty string:

```text
因此，

在开始使用守护程序集之前，通过将名称空间注释openshift.io/node-selector设置为空字符串，禁用名称空间中的默认项目范围节点选择器：
```

```text
# oc patch namespace myproject -p \
    '{"metadata": {"annotations": {"openshift.io/node-selector": ""}}}'
If you are creating a new project, overwrite the default node selector using oc adm new-project --node-selector="".
```

```text
#oc patch namespace myproject -p \
     '{“metadata”：{“annotations”：{“openshift.io/node-selector”：“”}}}'
如果要创建新项目，请使用oc adm new-project --node-selector =“”覆盖默认节点选择器。
```

## Creating Daemonsets

When creating daemonsets, the nodeSelector field is used to indicate the nodes on which the daemonset should deploy replicas.

```text
创建守护程序集时，nodeSelector字段用于指示守护程序应在其上部署副本的节点。
```

### Define the daemonset yaml file

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: hello-daemonset
spec:
  selector:
      matchLabels:
        name: hello-daemonset (1)
  template:
    metadata:
      labels:
        name: hello-daemonset (2)
    spec:
      nodeSelector: (3)
        type: infra
      containers:
      - image: openshift/hello-openshift
        imagePullPolicy: Always
        name: registry
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
      serviceAccount: default
      terminationGracePeriodSeconds: 10
```

(1)The label selector that determines which pods belong to the daemonset.
(2)The pod template’s label selector. Must match the label selector above.
(3)The node selector that determines on which nodes pod replicas should be deployed.

```text
（1）标签选择器，用于确定哪些pod属于守护进程。
（2）pod模板的标签选择器。 必须与上面的标签选择器匹配。
（3）节点选择器，用于确定应在哪些节点上部署pod副本。
```

### Create the daemonset object

```shell
oc create -f daemonset.yaml
```

To verify that the pods were created, and that each node has a pod replica:

```text
要验证是否已创建pod，并且每个节点都有一个pod副本：
```

* Find the daemonset pods:

```shell
$ oc get pods
hello-daemonset-cx6md   1/1       Running   0          2m
hello-daemonset-e3md9   1/1       Running   0          2m
```

* View the pods to verify the pod has been placed onto the node:

```shell
$ oc describe pod/hello-daemonset-cx6md|grep Node
Node:        openshift-node01.hostname.com/10.14.20.134
$ oc describe pod/hello-daemonset-e3md9|grep Node
Node:        openshift-node02.hostname.com/10.14.20.137
```

* If you update a DaemonSet’s pod template, the existing pod replicas are not affected.

* If you delete a DaemonSet and then create a new DaemonSet with a different template but the same label selector, it recognizes any existing pod replicas as having matching labels and thus does not update them or create new replicas despite a mismatch in the pod template.

* If you change node labels, the DaemonSet adds pods to nodes that match the new labels and deletes pods from nodes that do not match the new labels.

```text
*如果更新DaemonSet的pod模板，则现有pod副本不受影响。

*如果删除DaemonSet然后使用不同的模板但相同的标签选择器创建新的DaemonSet，它会将任何现有的pod副本识别为具有匹配的标签，因此尽管pod模板不匹配，但不会更新它们或创建新的副本。

*如果更改节点标签，DaemonSet会将pod添加到与新标签匹配的节点，并从与新标签不匹配的节点中删除pod。
```

To update a DaemonSet, force new pod replicas to be created by deleting the old replicas or nodes.

```text
要更新DaemonSet，请通过删除旧的副本或节点来强制创建新的pod副本。
```