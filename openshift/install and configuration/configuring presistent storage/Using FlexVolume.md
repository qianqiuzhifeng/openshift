# Persistent Storage Using FlexVolume Plug-ins

## Overview

OKD has built-in volume plug-ins to use different storage technologies. To use storage from a back-end that does not have a built-in plug-in, you can extend OKD through FlexVolume drivers and provide persistent storage to applications.

```text
OKD具有内置的卷插件，可以使用不同的存储技术。 要从没有内置插件的后端使用存储，您可以通过FlexVolume驱动程序扩展OKD并为应用程序提供持久存储。
```

## FlexVolume drivers

A FlexVolume driver is an executable file that resides in a well-defined directory on all machines in the cluster, both masters and nodes. OKD calls the FlexVolume driver whenever it needs to attach, detach, mount, or unmount a volume represented by a PersistentVolume with flexVolume as the source.

```text
FlexVolume驱动程序是一个可执行文件，驻留在集群中所有计算机上的明确定义的目录中，包括主服务器和节点。 只要需要使用flexVolume作为源来附加，分离，装载或卸载由PersistentVolume表示的卷，OKD就会调用FlexVolume驱动程序。
```

The first command-line argument of the driver is always an operation name. Other parameters are specific to each operation. Most of the operations take a JavaScript Object Notation (JSON) string as a parameter. This parameter is a complete JSON string, and not the name of a file with the JSON data.

```text
驱动程序的第一个命令行参数始终是操作名称。 其他参数特定于每个操作。 大多数操作都将JavaScript Object Notation（JSON）字符串作为参数。 此参数是完整的JSON字符串，而不是具有JSON数据的文件的名称。
```

The FlexVolume driver contains:

* All flexVolume.options.

* Some options from flexVolume prefixed by kubernetes.io/, such as fsType and readwrite.

* The content of the referenced secret, if specified, prefixed by kubernetes.io/secret/.

```text
FlexVolume驱动程序包含：

    所有flexVolume.options。

    flexVolume的一些选项以kubernetes.io/为前缀，例如fsType和readwrite。

    引用的secret的内容，如果指定，则以kubernetes.io/secret/为前缀。
```

FlexVolume driver JSON input example

```json
{
        "fooServer": "192.168.0.1:1234", (1)
        "fooVolumeName": "bar",
        "kubernetes.io/fsType": "ext4", (2)
        "kubernetes.io/readwrite": "ro", (3)
        "kubernetes.io/secret/<key name>": "<key value>", (4)
        "kubernetes.io/secret/<another key name>": "<another key value>",
}
```

(1)All options from flexVolume.options.
(2)The value of flexVolume.fsType.
(3)ro/rw based on flexVolume.readOnly.
(4)All keys and their values from the secret referenced by flexVolume.secretRef.

```text
（1）flexVolume.options的所有选项。
（2）flexVolume.fsType的值。
（3）基于flexVolume.readOnly的ro/rw。
（4）flexVolume.secretRef引用的秘密中的所有密钥及其值。
```

OKD expects JSON data on standard output of the driver. When not specified, the output describes the result of the operation.

```text
OKD期望JSON数据在驱动程序的标准输出上。 未指定时，输出描述操作的结果。
```

FlexVolume Driver Default Output

```json
{
        "status": "<Success/Failure/Not supported>",
        "message": "<Reason for success/failure>"
}
```

Exit code of the driver should be 0 for success and 1 for error.

```text
驱动程序的退出代码应为0表示成功，1表示错误。
```

Operations should be idempotent, which means that the attachment of an already attached volume or the mounting of an already mounted volume should result in a successful operation.

```text
操作应该是幂等的，这意味着已连接的卷的附件或已安装的卷的安装应该导致成功的操作。
```

The FlexVolume driver can work in two modes:

* with the master-initated attach/detach operation, or

* without the master-initated attach/detach operation.

```text
FlexVolume驱动程序可以在两种模式下工作：
    使用主发起的附加/分离操作，或
    没有主启动的附加/分离操作。
```

The attach/detach operation is used by the OKD master to attach a volume to a node and to detach it from a node. This is useful when a node becomes unresponsive for any reason. Then, the master can kill all pods on the node, detach all volumes from it, and attach the volumes to other nodes to resume the applications while the original node is still not reachable.

```text
OKD主机使用附加/分离操作将卷附加到节点并将其从节点分离。 当节点因任何原因无响应时，这很有用。 然后，主服务器可以终止节点上的所有pod，从中分离所有卷，并将卷附加到其他节点以恢复应用程序，同时仍无法访问原始节点。
```

Not all storage back-end supports master-initiated detachment of a volume from another machine.

```text
并非所有存储后端都支持从另一台计算机主控启动卷的分离。
```

## FlexVolume drivers with master-initiated attach/detach

A FlexVolume driver that supports master-controlled attach/detach must implement the following operations:

```text
支持主控制连接/分离的FlexVolume驱动程序必须实现以下操作：
```

### init

Initializes the driver. It is called during initialization of masters and nodes.

```text
初始化驱动器，在master和node初始化的时候调用
```

* Arguments: none

* Executed on: master, node

* Expected output: default JSON

```text
参数：无

执行于：master，node

预期输出：默认JSON
```

### getvolumename

Returns the unique name of the volume. This name must be consistent among all masters and nodes, because it is used in subsequent detach call as <volume-name>. Any / characters in the <volume-name> are automatically replaced by ~.

```text
返回卷的唯一名称。 此名称必须在所有主节点和节点之间保持一致，因为它在后续分离调用中用作<volume-name>。 <volume-name>中的任何/字符都会自动替换为〜。
```

* Arguments: <json>

* Executed on: master, node

* Expected output: default JSON + volumeName:

```json
{
        "status": "Success",
        "message": "",
        "volumeName": "foo-volume-bar" 
}
```

```text
参数：json

执行于：master,node

期望输出： 默认json + volumeName
```

The unique name of the volume in storage back-end foo.

```text
存储后端foo中卷的唯一名称。
```

### attach

Attaches a volume represented by the JSON to a given node. This operation should return the name of the device on the node if it is known, that is, if it has been assigned by the storage back-end before it runs. If the device is not known, the device must be found on the node by the subsequent waitforattach operation.

```text
将由JSON表示的卷附加到给定节点。 如果已知该操作，则该操作应返回节点上设备的名称，即，如果存储后端在运行之前已分配该设备的名称。 如果设备未知，则必须通过随后的waitforattach操作在节点上找到该设备。
```

* Arguments: <json> <node-name>

* Executed on: master

* Expected output: default JSON + device, if known:

```json
{
        "status": "Success",
        "message": "",
        "device": "/dev/xvda" 
}
```

```text
参数： json

执行于：master

期望输出：默认json + device
```

The name of the device on the node, if known.

### waitforattach

Waits until a volume is fully attached to a node and its device emerges. If the previous attach operation has returned <device-name>, it is provided as an input parameter. Otherwise, <device-name> is empty and the operation must find the device on the node.

```text
等待卷完全连接到节点并且其设备出现。 如果先前的附加操作已返回<device-name>，则将其作为输入参数提供。 否则，<device-name>为空，操作必须在节点上找到该设备。
```

* Arguments: <device-name> <json>

* Executed on: node

* Expected output: default JSON + device

```json
{
        "status": "Success",
        "message": "",
        "device": "/dev/xvda" 
}
```

```text
参数： device-name json

执行于：node

期望输出：默认json + device
```

The name of the device on the node.

### detach

Detaches the given volume from a node. <volume-name> is the name of the device returned by the getvolumename operation. Any / characters in the <volume-name> are automatically replaced by ~.

```text
从节点中分离给定的卷。 <volume-name>是getvolumename操作返回的设备的名称。 <volume-name>中的任何/字符都会自动替换为〜。
```

* Arguments: <volume-name> <node-name>

* Executed on: master

* Expected output: default JSON

```text
参数： volume-name> node-name

执行于： master

期望输出： 默认json
```

### isattached

Checks that a volume is attached to a node.

```text
确认数据卷是否挂载于node
```

* Arguments: <json> <node-name>

* Executed on: master

* Expected output: default JSON + attached

```json
{
        "status": "Success",
        "message": "",
        "attached": true (1)
}
```

(1)The status of attachment of the volume to the node.

### mountdevice

Mounts a volume’s device to a directory. <device-name> is name of the device as returned by the previous waitforattach operation.

```text
将卷的设备安装到目录。 <device-name>是先前waitforattach操作返回的设备名称。
```

* Arguments: <mount-dir> <device-name> <json>

* Executed on: node

* Expected output: default JSON

```text
参数： <mount-dir> <device-name> <json>

执行于：node

期望输出：默认json
```

### unmountdevice

Unmounts a volume’s device from a directory.

```text
从目录中卸载卷的设备。
```

* Arguments: <mount-dir>

* Executed on: node

```text
参数：  <mount-dir>

执行于：node
```

All other operations should return JSON with {"status": "Not supported"} and exit code 1.

```text
所有其他操作应返回带有{“status”：“Not supported”}的JSON并退出代码1。
```

Master-initiated attach/detach operations are enabled by default in OKD 3.6. They may work in older versions, but must be explicitly enabled. See Enabling Controller-managed Attachment and Detachment. When not enabled, the attach/detach operations are initiated by a node where the volume should be attached to or detached from. Syntax and all parameters of FlexVolume driver invocations are the same in both cases.

```text
默认情况下，在OKD 3.6中启用主发起的附加/分离操作。 它们可能在旧版本中工作，但必须明确启用。 请参阅启用Controller管理的附件和分离。 未启用时，附加/分离操作由应连接或分离卷的节点启动。 两种情况下，FlexVolume驱动程序调用的语法和所有参数都相同。
```

## FlexVolume drivers without master-initiated attach/detach

FlexVolume drivers that do not support master-controlled attach/detach are executed only on the node and must implement these operations:

```text
不支持主控制连接/分离的FlexVolume驱动程序仅在节点上执行，并且必须实现以下操作：
```

### init

Initializes the driver. It is called during initialization of all nodes.

```text
初始化驱动程序。 在初始化所有节点期间调用它。
```

* Arguments: none

* Executed on: node

* Expected output: default JSON

```text
参数： none

执行于： node

期望输出： 默认json
```

### mount

Mounts a volume to directory. This can include anything that is necessary to mount the volume, including attaching the volume to the node, finding the its device, and then mounting the device.

```text
将卷安装到目录。 这可以包括安装卷所需的任何内容，包括将卷附加到节点，查找其设备，然后安装设备。
```

* Arguments: <mount-dir> <json>

* Executed on: node

* Expected output: default JSON

```text
参数：mount-dir json

执行于：node

期望输出：默认json
```

### unmount

Unmounts a volume from a directory. This can include anything that is necessary to clean up the volume after unmounting, such as detaching the volume from the node.

```text
从目录中卸载卷。 这可以包括卸载后清理卷所需的任何内容，例如从节点中分离卷。
```

* Arguments: <mount-dir>

* Executed on: node

* Expected output: default JSON

```text
参数： <mount-dir>

执行于： node

期望输出： 默认json
```

All other operations should return JSON with {"status": "Not supported"} and exit code 1.

## Installing FlexVolume drivers

To install the FlexVolume driver:

* Ensure that the executable file exists on all masters and nodes in the cluster.

* Place the executable file at the volume plug-in path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec/<vendor>~<driver>/<driver>.

```text
要安装FlexVolume驱动程序：

*确保可执行文件存在于群集中的所有主服务器和节点上。

*将可执行文件放在卷插件路径中：/usr/libexec/kubernetes/kubelet-plugins/volume/exec/<vendor>〜<driver>/<driver>。
```

For example, to install the FlexVolume driver for the storage foo, place the executable file at: /usr/libexec/kubernetes/kubelet-plugins/volume/exec/openshift.com~foo/foo.

```text
例如，要为存储foo安装FlexVolume驱动程序，请将可执行文件放在：/usr/libexec/kubernetes/kubelet-plugins/volume/exec/openshift.com~foo/foo。
```

## Consuming storage using FlexVolume drivers

Use the PersistentVolume object to reference the installed storage. Each PersistentVolume object in OKD represents one storage asset, typically a volume, in the storage back-end.

```text
使用PersistentVolume对象引用已安装的存储。 OKD中的每个PersistentVolume对象代表存储后端中的一个存储资产，通常是卷。
```

Persistent volume object definition using FlexVolume drivers example

```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
        name: pv0001 (1)
    spec:
        capacity:
            storage: 1Gi (2)
        accessModes:
            - ReadWriteOnce
        flexVolume:
            driver: openshift.com/foo (3)
            fsType: "ext4" (4)
            secretRef: foo-secret (5)
            readOnly: true (6)
            options: (7)
            fooServer: 192.168.0.1:1234
            fooVolumeName: bar
```

(1)The name of the volume. This is how it is identified through persistent volume claims or from pods. This name can be different from the name of the volume on back-end storage.
(2)The amount of storage allocated to this volume.
(3)The name of the driver. This field is mandatory.
(4)The file system that is present on the volume. This field is optional.
(5)The reference to a secret. Keys and values from this secret are provided to the FlexVolume driver on invocation. This field is optional.
(6)The read-only flag. This field is optional.
(7)The additional options for the FlexVolume driver. In addition to the flags specified by the user in the options field, the following flags are also passed to the executable:

```text
（1）卷的名称。 这是通过持久量声明或从pod中识别它的方式。 此名称可以与后端存储上的卷名称不同。
（2）分配给该卷的存储量。
（3）驱动的名字。 此字段是必填字段。
（4）卷上存在的文件系统。 该字段是可选的。
（5）提到秘密。 来自此密钥的密钥和值将在调用时提供给FlexVolume驱动程序。 该字段是可选的。
（6）只读标志。 该字段是可选的。
（7）FlexVolume驱动程序的附加选项。 除了用户在options字段中指定的标志之外，还将以下标志传递给可执行文件：
```

```json
"fsType":"<FS type>",
"readwrite":"<rw>",
"secret/key1":"<secret1>"
...
"secret/keyN":"<secretN>"
```