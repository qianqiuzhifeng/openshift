# DeploymentConfig

## Deployment Strategies(部署策略)

A deployment strategy is a way to change or upgrade an application. The aim is to make the change without downtime in a way that the user barely notices the improvements.

```text
部署策略是一种更改或升级应用程序的方法。 目的是在不停机的情况下以用户几乎没有注意到改进的方式进行更改。
```

The most common strategy is to use a blue-green deployment. The new version (the blue version) is brought up for testing and evaluation, while the users still use the stable version (the green version). When ready, the users are switched to the blue version. If a problem arises, you can switch back to the green version.

```text
最常见的策略是使用蓝绿色部署。 新版本（蓝色版本）用于测试和评估，而用户仍然使用稳定版本（绿色版本）。 准备好后，用户将切换到蓝色版本。 如果出现问题，您可以切换回绿色版本。
```

A common alternative strategy is to use A/B versions that are both active at the same time and some users use one version, and some users use the other version. This can be used for experimenting with user interface changes and other features to get user feedback. It can also be used to verify proper operation in a production context where problems impact a limited number of users.

```text
一种常见的替代策略是使用同时处于活动状态的A/B版本，而某些用户使用一个版本，而某些用户使用另一个版本。 这可用于试验用户界面更改和其他功能以获得用户反馈。 它还可用于验证生产环境中的正确操作，其中问题会影响有限数量的用户。
```

A canary deployment tests the new version but when a problem is detected it quickly falls back to the previous version. This can be done with both of the above strategies.

```text
金丝雀部署测试新版本，但是当检测到问题时，它会迅速回退到先前版本。 这可以通过上述两种策略来完成。
```

The route based deployment strategies do not scale the number of pods in the services. To maintain desired performance characteristics the deployment configurations may need to be scaled.

```text
基于路由的部署策略不会扩展服务中的pod数量。 为了保持期望的性能特征，可能需要缩放部署配置。
```

There are things to consider when choosing a deployment strategy.

* Long running connections need to be handled gracefully.

* Database conversions can get tricky and will need to be done and rolled back along with the application.

* If the application is a hybrid of microservices and traditional components downtime may be needed to complete the transition.

* You need the infrastructure to do this.

* If you have a non-isolated test environment, you can break both new and old versions.

```text
选择部署策略时需要考虑的事项。

    需要优雅地处理长时间运行的连接。

    数据库转换可能会变得棘手，需要与应用程序一起完成并回滚。

    如果应用程序是微服务和传统组件的混合，则可能需要停机时间来完成转换。

    您需要基础架构才能执行此操作。

    如果您有一个非隔离的测试环境，则可以同时破坏新旧版本。
```

Since the end user usually accesses the application through a route handled by a router, the deployment strategy can focus on deployment configuration features or routing features.

```text
由于最终用户通常通过路由器处理的路由访问应用程序，因此部署策略可以专注于部署配置功能或路由功能。
```

Strategies that focus on the deployment configuration impact all routes that use the application. Strategies that use router features target individual routes.

```text
专注于部署配置的策略会影响使用该应用程序的所有路由。 使用路由器功能的策略针对单个路由。
```

Many deployment strategies are supported through the deployment configuration and some additional strategies are supported through router features. The deployment configuration-based strategies are discussed in this section.

* Rolling Strategy and Canary Deployments

* Recreate Strategy

* Custom Strategy

* Blue-Green Deployment using routes

* A/B Deployment and canary deployments using routes

* One Service, Multiple Deployment Configurations

```text
部署配置支持许多部署策略，并且通过路由器功能支持一些其他策略。 本节将讨论基于部署配置的策略。

    滚动策略和金丝雀部署

    重建策略

    定制策略

    使用路线进行蓝绿部署

    使用路由进行A/B部署和canary部署

    一个服务，多个部署配置
```

The Rolling strategy is the default strategy used if no strategy is specified on a deployment configuration.

```text
如果在部署配置中未指定策略，则Rolling策略是使用的默认策略。
```

A deployment strategy uses readiness checks to determine if a new pod is ready for use. If a readiness check fails, the deployment configuration will retry to run the pod until it times out. The default timeout is 10m, a value set in TimeoutSeconds in dc.spec.strategy.*params.

```text
部署策略使用准备情况检查来确定新pod是否可以使用。 如果准备情况检查失败，则部署配置将重试运行容器，直到超时为止。 默认超时为10m，在dc.spec.strategy。* params中的TimeoutSeconds中设置的值。
```

## Rolling Strategy(滚动策略)

A rolling deployment slowly replaces instances of the previous version of an application with instances of the new version of the application. A rolling deployment typically waits for new pods to become ready via a readiness check before scaling down the old components. If a significant issue occurs, the rolling deployment can be aborted.

```text
滚动部署会慢慢使用新版本应用程序的实例替换先前版本的应用程序的实例。滚动部署通常会在缩小旧组件之前等待新的pod通过准备情况检查准备就绪。如果发生重大问题，则可以中止滚动部署。
```

### Canary Deployments(金丝雀部署)

All rolling deployments in OKD are canary deployments; a new version (the canary) is tested before all of the old instances are replaced. If the readiness check never succeeds, the canary instance is removed and the deployment configuration will be automatically rolled back. The readiness check is part of the application code, and may be as sophisticated as necessary to ensure the new instance is ready to be used. If you need to implement more complex checks of the application (such as sending real user workloads to the new instance), consider implementing a custom deployment or using a blue-green deployment strategy.

```text
OKD中的所有滚动部署都是canary部署;在替换所有旧实例之前测试新版本（金丝雀）。如果准备检查永远不会成功，则会删除canary实例，并自动回滚部署配置。准备情况检查是应用程序代码的一部分，并且可以根据需要进行复杂化以确保准备好使用新实例。如果需要对应用程序执行更复杂的检查（例如将实际用户工作负载发送到新实例），请考虑实施自定义部署或使用蓝绿部署策略。
```

### When to Use a Rolling Deployment(什么时候使用滚动策略)

* When you want to take no downtime during an application update.

* When your application supports having old code and new code running at the same time.

```text
如果您希望在应用程序更新期间不停机。

当您的应用程序支持同时运行旧代码和新代码时。
```

A rolling deployment means you to have both old and new versions of your code running at the same time. This typically requires that your application handle N-1 compatibility.

```text
滚动部署意味着您可以同时运行旧版本和新版本的代码。这通常要求您的应用程序处理N-1兼容性。
```

The following is an example of the Rolling strategy:

    strategy:
        type: Rolling
        rollingParams:
            updatePeriodSeconds: 1 （1）
            intervalSeconds: 1 （2）
            timeoutSeconds: 120 （3）
            maxSurge: "20%" （4）
            maxUnavailable: "10%" （5）
            pre: {} （6）
            post: {}

（1）The time to wait between individual pod updates. If unspecified, this value defaults to 1.
（2）The time to wait between polling the deployment status after update. If unspecified, this value defaults to 1.
（3）The time to wait for a scaling event before giving up. Optional; the default is 600. Here, giving up means automatically rolling back to the previous complete deployment.
（4）maxSurge is optional and defaults to 25% if not specified. See the information below the following procedure.
（5）maxUnavailable is optional and defaults to 25% if not specified. See the information below the following procedure.
（6）pre and post are both lifecycle hooks.

```text
下面是一个滚动策略：
    strategy: # 策略
        type: Rolling # 滚动
        rollingParams: # 滚动参数
            updatePeriodSeconds: 1 （1）
            intervalSeconds: 1 （2）
            timeoutSeconds: 120 （3）
            maxSurge: "20%" （4）
            maxUnavailable: "10%" （5）
            pre: {} （6）
            post: {}

（1）各个pod更新之间的等待时间。 如果未指定，则此值默认为1。
（2）更新后轮询部署状态之间的等待时间。 如果未指定，则此值默认为1。
（3）在放弃之前等待缩放事件的时间。 可选的; 默认值为600.此处，放弃意味着自动回滚到之前的完整部署。
（4）maxSurge是可选的，如果未指定，则默认为25％。 请参阅以下过程中的信息。
（5）maxUnavailable是可选的，如果未指定，则默认为25％。 请参阅以下过程中的信息。
（6）pre和post都是生命周期钩子。
```

The Rolling strategy will:

1. Execute any pre lifecycle hook.

2. Scale up the new replication controller based on the surge count.

3. Scale down the old replication controller based on the max unavailable count.

4. Repeat this scaling until the new replication controller has reached the desired replica count and the old replication controller has been scaled to zero.

5. Execute any post lifecycle hook.

```text滚动策略将：

执行任何pre生命周期钩子。

根据浪涌计数扩展新的复制控制器。

根据最大不可用计数缩小旧复制控制器。

重复此缩放，直到新的复制控制器达到所需的副本计数并且旧的复制控制器已缩放为零。

执行任何post生命周期钩子。
```

When scaling down, the Rolling strategy waits for pods to become ready so it can decide whether further scaling would affect availability. If scaled up pods never become ready, the deployment process will eventually time out and result in a deployment failure.

```text
按比例缩小时，Rolling策略会等待pod准备就绪，因此可以决定进一步缩放是否会影响可用性。 如果扩展的pod从未准备就绪，则部署过程最终会超时并导致部署失败。
```

The maxUnavailable parameter is the maximum number of pods that can be unavailable during the update. The maxSurge parameter is the maximum number of pods that can be scheduled above the original number of pods. Both parameters can be set to either a percentage (e.g., 10%) or an absolute value (e.g., 2). The default value for both is 25%.

```text
maxUnavailable参数是更新期间可用的最大pod数。maxSurge参数是可以在原始pod数量之上调度的最大pod数。两个参数可以设置为百分比（例如，10％）或绝对值（例如，2）。两者的默认值均为25％。
```

These parameters allow the deployment to be tuned for availability and speed. For example:

* maxUnavailable=0 and maxSurge=20% ensures full capacity is maintained during the update and rapid scale up.

* maxUnavailable=10% and maxSurge=0 performs an update using no extra capacity (an in-place update).

* maxUnavailable=10% and maxSurge=10% scales up and down quickly with some potential for capacity loss.

```text
这些参数允许调整部署的可用性和速度。 例如：

    maxUnavailable = 0和maxSurge = 20％确保在更新和快速扩展期间保持满容量。

    maxUnavailable = 10％且maxSurge = 0使用无额外容量执行更新（就地更新）。

    maxUnavailable = 10％和maxSurge = 10％快速扩大和缩小，可能会有一些容量损失。
```

Generally, if you want fast rollouts, use maxSurge. If you need to take into account resource quota and can accept partial unavailability, use maxUnavailable.

```text
通常，如果要快速推出，请使用maxSurge。 如果您需要考虑资源配额并且可以接受部分不可用，请使用maxUnavailable。
```

### Rolling Example(滚动示例)

Rolling deployments are the default in OKD. To see a rolling update, follow these steps:

1. Create an application based on the example deployment images found in DockerHub:

```shell
oc new-app openshift/deployment-example
```

* If you have the router installed, make the application available via a route (or use the service IP directly)

```shell
oc expose svc/deployment-example
```

* Browse to the application at deployment-example.<project>.<router_domain> to verify you see the v1 image.

2. Scale the deployment configuration up to three replicas:

```shell
oc scale dc/deployment-example --replicas=3
```

3. Trigger a new deployment automatically by tagging a new version of the example as the latest tag:

```shell
oc tag deployment-example:v2 deployment-example:latest
```

4. In your browser, refresh the page until you see the v2 image.

5. If you are using the CLI, the following command will show you how many pods are on version 1 and how many are on version 2. In the web console, you should see the pods slowly being added to v2 and removed from v1.

```shell
oc describe dc deployment-example
```

During the deployment process, the new replication controller is incrementally scaled up. Once the new pods are marked as ready (by passing their readiness check), the deployment process will continue. If the pods do not become ready, the process will abort, and the deployment configuration will be rolled back to its previous version.

```text
在部署过程中，新的复制控制器将逐步扩展。 一旦新的pod被标记为就绪（通过准备就绪检查），部署过程将继续。 如果窗格未准备就绪，则该过程将中止，并且部署配置将回滚到其先前版本。
```

## Recreate Strategy(重建策略)

The Recreate strategy has basic rollout behavior and supports lifecycle hooks for injecting code into the deployment process.

```text
重建策略具有基本的部署行为，并支持生命周期钩子，用于将代码注入部署过程。
```

The following is an example of the Recreate strategy:

strategy:
  type: Recreate
  recreateParams:（1）
    pre: {} （2）
    mid: {}
    post: {}
（1）recreateParams are optional.
（2）pre, mid, and post are lifecycle hooks.

```text
以下是重新创建策略的示例：
    strategy:
    type: Recreate
    recreateParams:（1）
        pre: {} （2）
        mid: {}
        post: {}
（1）recreateParams是可选的。
（2）pre，mid和post是生命周期钩子。
```

The Recreate strategy will:

* Execute any pre lifecycle hook.

* Scale down the previous deployment to zero.

* Execute any mid lifecycle hook.

* Scale up the new deployment.

* Execute any post lifecycle hook.

```text
重建策略将：

    执行任何pre生命周期钩子。

    将先前部署缩小为零。

    执行任何中期生命周期钩子。

    扩展新部署。

    执行任何post生命周期钩子。
```

During scale up, if the replica count of the deployment is greater than one, the first replica of the deployment will be validated for readiness before fully scaling up the deployment. If the validation of the first replica fails, the deployment will be considered a failure.

```text
在扩展期间，如果部署的副本计数大于1，则在完全扩展部署之前，将验证部署的第一个副本的准备情况。 如果第一个副本的验证失败，则部署将被视为失败。
```

When to Use a Recreate Deployment

* When you must run migrations or other data transformations before your new code starts.

* When you do not support having new and old versions of your application code running at the same time.

* When you want to use a RWO volume, which is not supported being shared between multiple replicas.

```text
何时使用重新创建部署:
    在新代码启动之前必须运行迁移或其他数据转换时。

    当您不支持同时运行新旧版本的应用程序代码时。

    如果要使用不支持在多个副本之间共享的RWO卷。
```

A recreate deployment incurs downtime because, for a brief period, no instances of your application are running. However, your old code and new code do not run at the same time.

```text
重新创建部署会导致停机，因为在短时间内，您的应用程序的任何实例都没有运行。 但是，旧代码和新代码不会同时运行。
```

## Custom Strategy(定制策略)

The Custom strategy allows you to provide your own deployment behavior.

```text
定制策略允许您提供自己的部署行为。
```

The following is an example of the Custom strategy:

    strategy:
    type: Custom
    customParams:
        image: organization/strategy
        command: [ "command", "arg1" ]
        environment:
        - name: ENV_1
            value: VALUE_1

In the above example, the organization/strategy container image provides the deployment behavior. The optional command array overrides any CMD directive specified in the image’s Dockerfile. The optional environment variables provided are added to the execution environment of the strategy process.

```text
在上面的示例中，组织/策略容器映像提供部署行为。 可选的命令数组会覆盖映像的Dockerfile中指定的任何CMD指令。 提供的可选环境变量将添加到策略流程的执行环境中。
```

Additionally, OKD provides the following environment variables to the deployment process:

|Environment Variable | Description|

OPENSHIFT_DEPLOYMENT_NAME | The name of the new deployment (a replication controller).

OPENSHIFT_DEPLOYMENT_NAMESPACE | The name space of the new deployment.

```text
此外，OKD还为部署过程提供以下环境变量：

|环境变量|说明|

OPENSHIFT_DEPLOYMENT_NAME | 新部署的名称（复制控制器）。

OPENSHIFT_DEPLOYMENT_NAMESPACE | 新部署的名称空间。

```

The replica count of the new deployment will initially be zero. The responsibility of the strategy is to make the new deployment active using the logic that best serves the needs of the user.

```text
新部署的副本计数最初为零。 该策略的职责是使用最能满足用户需求的逻辑使新部署处于活动状态。
```

Alternatively, use customParams to inject the custom deployment logic into the existing deployment strategies. Provide a custom shell script logic and call the openshift-deploy binary. Users do not have to supply their custom deployer container image, but the default OKD deployer image will be used instead:

    strategy:
    type: Rolling
    customParams:
        command:
        - /bin/sh
        - -c
        - |
        set -e
        openshift-deploy --until=50%
        echo Halfway there
        openshift-deploy
        echo Complete

```text
或者，使用customParams将自定义部署逻辑注入现有部署策略。 提供自定义shell脚本逻辑并调用openshift-deploy二进制文件。 用户不必提供他们的自定义部署容器映像，但将使用默认的OKD部署者映像：
```

This will result in following deployment:

    Started deployment #2
    --> Scaling up custom-deployment-2 from 0 to 2, scaling down custom-deployment-1 from 2 to 0 (keep 2 pods available, don't exceed 3 pods)
        Scaling custom-deployment-2 up to 1
    --> Reached 50% (currently 50%)
    Halfway there
    --> Scaling up custom-deployment-2 from 1 to 2, scaling down custom-deployment-1 from 2 to 0 (keep 2 pods available, don't exceed 3 pods)
        Scaling custom-deployment-1 down to 1
        Scaling custom-deployment-2 up to 2
        Scaling custom-deployment-1 down to 0
    --> Success
    Complete

If the custom deployment strategy process requires access to the OKD API or the Kubernetes API the container that executes the strategy can use the service account token available inside the container for authentication.

```text
如果自定义部署策略进程需要访问OKD API或Kubernetes API，则执行策略的容器可以使用容器内可用的服务帐户令牌进行身份验证。
```

## Lifecycle Hooks(生命周期钩子)

The Recreate and Rolling strategies support lifecycle hooks, which allow behavior to be injected into the deployment process at predefined points within the strategy:

```text
重新创建和滚动策略支持生命周期钩子，允许在策略中的预定义点将行为注入到部署过程中：
```

The following is an example of a pre lifecycle hook:

    pre:
        failurePolicy: Abort
        execNewPod: {} (1)

(1)execNewPod is a pod-based lifecycle hook.

Every hook has a failurePolicy, which defines the action the strategy should take when a hook failure is encountered:

Abort The deployment process will be considered a failure if the hook fails.

Retry The hook execution should be retried until it succeeds.

Ignore Any hook failure should be ignored and the deployment should proceed.

Hooks have a type-specific field that describes how to execute the hook. Currently, pod-based hooks are the only supported hook type, specified by the execNewPod field.

```text
每个钩子都有一个failurePolicy，它定义了遇到钩子失败时策略应该采取的操作：

中止 如果挂钩失败，则部署过程将被视为失败。

重试 应重试挂钩执行，直到成功为止。

忽略 任何挂钩失败都应该被忽略，部署应该继续进行。

挂钩具有特定于类型的字段，用于描述如何执行挂钩。 目前，基于pod的挂钩是唯一受支持的挂钩类型，由execNewPod字段指定。
```

### Pod-based Lifecycle Hook（基于Pod的生命周期钩子）

Pod-based lifecycle hooks execute hook code in a new pod derived from the template in a deployment configuration.

```text
基于Pod的生命周期钩子在部署配置中从模板派生的新pod中执行钩子代码。
```

The following simplified example deployment configuration uses the Rolling strategy. Triggers and some other minor details are omitted for brevity:

```text
以下简化示例部署配置使用Rolling策略。 为简洁起见，省略了触发器和一些其他细节：
```

    kind: DeploymentConfig
    apiVersion: v1
    metadata:
    name: frontend
    spec:
    template:
        metadata:
        labels:
            name: frontend
        spec:
        containers:
            - name: helloworld
            image: openshift/origin-ruby-sample
    replicas: 5
    selector:
        name: frontend
    strategy:
        type: Rolling
        rollingParams:
        pre:
            failurePolicy: Abort
            execNewPod:
            containerName: helloworld (1)
            command: [ "/usr/bin/command", "arg1", "arg2" ] (2)
            env: (3)
                - name: CUSTOM_VAR1
                value: custom_value1
            volumes:
                - data (4)

(1)The helloworld name refers to spec.template.spec.containers[0].name.

(2)This command overrides any ENTRYPOINT defined by the openshift/origin-ruby-sample image.

(3)env is an optional set of environment variables for the hook container.

(4)volumes is an optional set of volume references for the hook container

```text
(1)helloworld名称是指spec.template.spec.containers[0].name。
(2)此命令将覆盖openshift/origin-ruby-sample映像定义的任何ENTRYPOINT。
(3)env是钩子容器的一组可选环境变量。
(4)volumes是钩子容器的一组可选卷引用
```

In this example, the pre hook will be executed in a new pod using the openshift/origin-ruby-sample image from the helloworld container. The hook pod will have the following properties:

* The hook command will be /usr/bin/command arg1 arg2.

* The hook container will have the CUSTOM_VAR1=custom_value1 environment variable.

* The hook failure policy is Abort, meaning the deployment process will fail if the hook fails.

* The hook pod will inherit the data volume from the deployment configuration pod.

```text
在此示例中，将使用helloworld容器中的openshift/origin-ruby-sample映像在新窗格中执行预挂钩。 挂钩吊舱将具有以下属性：

    hook命令将是/usr/bin/命令arg1 arg2。

    钩子容器将具有CUSTOM_VAR1 = custom_value1环境变量。

    挂钩失败策略是Abort，这意味着如果挂钩失败，部署过程将失败。

    挂钩窗格将从部署配置窗格继承数据卷。
```

### Using the Command Line(使用命令行)

The oc set deployment-hook command can be used to set the deployment hook for a deployment configuration. For the example above, you can set the pre-deployment hook with the following command:

```text
oc set deployment-hook命令可用于设置部署配置的部署挂钩。 对于上面的示例，您可以使用以下命令设置预部署挂钩：
```

```shell
oc set deployment-hook dc/frontend --pre -c helloworld -e CUSTOM_VAR1=custom_value1 \
  -v data --failure-policy=abort -- /usr/bin/command arg1 arg2
```