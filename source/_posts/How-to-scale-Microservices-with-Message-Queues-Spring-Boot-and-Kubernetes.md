---
title: 如何使用消息队列、Spring Boot 和 Kubernetes 扩展微服务
date: 2022-03-24 11:03:50
categories:
- SpringBoot
- Java
tags:
- SprintBoot
- Java
- Kubernetes
---


{% asset_img 1.png 示意图 width="400" %}

当您大规模设计和构建应用程序时，您将面临两个重大挑战：可扩展性和稳健性。
您应该设计您的服务，以便即使它受到间歇性重负载的影响，它也能继续可靠地运行。
以 Apple Store 为例。
每年都有数百万 Apple 客户预先注册购买新 iPhone。
数百万人同时购买一件商品。

<!--more-->

如果您将 Apple 商店的流量描绘为随时间变化的每秒请求数，图表可能如下所示：

{% asset_img 2.png 示意图 width="400" %}

现在想象一下，你的任务是构建这样的应用程序。
您正在建立一个商店，用户可以在其中购买他们最喜欢的商品。
您构建一个微服务来呈现网页并提供静态资产。 您还构建了一个后端 REST API 来处理传入的请求。
您希望将这两个组件分开，因为使用相同的 REST API，您可以为网站和移动应用程序提供服务。

{% asset_img 3.gif 示意图 width="400" %}

(开始想象)
今天是大日子，你的商店上线了。
您决定将应用程序扩展到四个前端实例和四个后端实例，因为您预测该网站将比平时更繁忙。

{% asset_img 4.gif 示意图 width="400" %}

您开始收到越来越多的流量。
前端服务正在处理流量。 但是您注意到连接到数据库的后端正在努力跟上事务的数量。
不用担心，您可以将后端的副本数扩展到 8 个。

{% asset_img 5.gif 示意图 width="400" %}

您收到的流量更多了，而后端却无法应对。
一些服务开始断开连接。 愤怒的客户开始联系你的客服抱（liao）怨（tian）。 
您的后端越来越乏力，并且丢失了大量的连接。

{% asset_img 6.gif 示意图 width="400" %}

你为此损失了一大笔钱，而你的客户则很不高兴。

您的应用程序并非设计为健壮和高可用性：
 * 前端和后端是紧密耦合的——事实上它不能处理没有后端的应用程序
 * 前端和后端必须协同扩展——如果没有足够的后端，你可能会淹没在流量中
 * 如果后端不可用，您将无法处理传入交易。

交易损失就是收入损失。
所以，你不（zao）得（gan）不（ma）（qu le）重新设计你的架构：用队列解耦前端和后端。

{% asset_img 7.gif 示意图 width="400" %}

前端将消息发布到队列，而后端同时处理一条待处理的消息。
新架构有一些明显的好处：
 * 如果后端不可用，则队列充当缓冲区
 * 如果前端产生的消息多于后端可以处理的消息，则这些消息将缓冲在队列中
 * 您可以独立于前端扩展后端 - 即您可以拥有数百个前端服务和一个后端实例
太好了，但是您如何构建这样的应用程序？
您如何设计可以处理数十万个请求的服务？ 
以及如何部署动态扩展的应用程序？
在深入了解部署和扩展的细节之前，让我们先关注应用程序。

## 编写 Spring 应用程序

该服务包含三个组件：前端、后端和消息代理。
前端是一个带有 Thymeleaf 模板引擎的简单 Spring Boot Web 应用程序。
后端是一个从队列中消费消息的工作人员。
由于 [Spring Boot 与 JMS 具有出色的集成](https://spring.io/guides/gs/messaging-jms/)，您可以使用它来发送和接收异步消息。
您可以在 learnk8s/spring-boot-k8s-hpa 找到一个包含连接到 JMS 的前端和后端应用程序的示例项目。

> 请注意，该应用程序是用 Java 10 编写的，以利用改进的 Docker 容器集成。

只需要一个代码库，您可以将项目配置为作为前端或后端运行。
该应用程序具有：
 * 可以购买商品的主页
 * 一个管理面板，您可以在其中检查队列中的消息数量
 * 一个 /health 端点，用于在应用程序准备好接收流量时发出信号
 * 一个 /submit 端点，它接收来自表单的提交并在队列中创建消息
 * 一个 /metrics 端点，用于公开队列中待处理消息的数量（稍后会详细介绍）
该应用程序可以在两种模式下运行：
**作为前端**，应用程序呈现人们可以购买商品的网页。

{% asset_img 8.png 示意图 width="400" %}

作为Worker，应用程序等待队列中的消息并处理它们。

{% asset_img 9.png 示意图 width="400" %}

> 请注意，在示例项目中，处理是通过使用 Thread.sleep(5000) 等待五秒钟来模拟的。

您可以通过更改 application.yaml 中的值来在任一模式下配置应用程序。

## 让项目程序先空转一会儿

默认情况下，应用程序作为前端和Worker启动。
您可以运行该应用程序，并且只要您有一个本地运行的 ActiveMQ 实例，您应该能够购买物品并让系统处理这些物品。

{% asset_img 10.gif 示意图 width="400" %}

如果您检查日志，您应该会看到worker正在处理项目。
完美！ 编写 Spring Boot 应用程序很容易。
一个更有趣的主题是学习如何将 Spring Boot 连接到消息代理。

## 使用 JMS 发送和接收消息
Spring JMS（Java 消息服务）是一种使用标准协议发送和接收消息的强大机制。
如果您过去使用过 JDBC API，您应该会发现 JMS API 很熟悉，因为它的工作方式类似。
您可以与 JMS 一起使用的最流行的消息代理是 ActiveMQ — 一个开源消息传递服务器。
使用这两个组件，您可以使用熟悉的接口 (JMS) 将消息发布到队列 (ActiveMQ)，并使用相同的接口接收消息。
更棒的是，Spring Boot 与 JMS 具有出色的集成，因此您可以立即上手。
实际上，下面的短类封装了用于与队列交互的逻辑：

```
@Component
public class QueueService implements MessageListener {
private static final Logger LOGGER = LoggerFactory.getLogger(QueueService.class);
@Autowired
  private JmsTemplate jmsTemplate;
  public void send(String destination, String message) {
    LOGGER.info("sending message='{}' to destination='{}'", message, destination);
    jmsTemplate.convertAndSend(destination, message);
  }
@Override
  public void onMessage(Message message) {
    if (message instanceof ActiveMQTextMessage) {
      ActiveMQTextMessage textMessage = (ActiveMQTextMessage) message;
      try {
        LOGGER.info("Processing task " + textMessage.getText());
        Thread.sleep(5000);
        LOGGER.info("Completed task " + textMessage.getText());
      } catch (InterruptedException e) {
        e.printStackTrace();
      } catch (JMSException e) {
        e.printStackTrace();
      }
    } else {
      LOGGER.error("Message is not a text message " + message.toString());
    }
  }
}
```

您可以使用 send 方法将消息发布到命名队列。
此外，Spring Boot 将为每条传入消息执行 onMessage 方法。
最后一块内容要说的是指示 Spring Boot 使用该类。
您可以通过在 [Spring Boot 应用程序中注册侦听器](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#jms-annotated-programmatic-registration)来在后台处理消息，如下所示：

```
@SpringBootApplication
@EnableJms
public class SpringBootApplication implements JmsListenerConfigurer {
  @Autowired
  private QueueService queueService;
public static void main(String[] args) {
    SpringApplication.run(SpringBootApplication.class, args);
  }
@Override
  public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
    SimpleJmsListenerEndpoint endpoint = new SimpleJmsListenerEndpoint();
    endpoint.setId("myId");
    endpoint.setDestination("queueName");
    endpoint.setMessageListener(queueService);
    registrar.registerEndpoint(endpoint);
  }
}
```

其中 id 是消费者的唯一标识符，destination 是队列的名称。
请注意您是如何用不到 40 行代码编写出可靠队列的。
你一定会爱上 Spring Boot。

## 节省部署的所有时间都可以专注于编码

您验证了应用程序工作正常，终于到了部署它的时候了。
此时，您可以启动您的 VPS，安装 Tomcat，并花一些时间编写自定义脚本来测试、构建、打包和部署应用程序。
或者您可以写下您希望拥有的内容的描述：一个消息代理和两个使用负载均衡器部署的应用程序。
Kubernetes 等编排器可以读取您的清单并配置正确基础架构。
由于在基础架构上花费的时间越少，编码时间就越多，这次您将把应用程序部署到 Kubernetes。但在开始之前，您需要一个 Kubernetes 集群。
您可以注册 Google Cloud Platform 或 Azure，并使用 Kubernetes 提供的云提供商。或者，您可以在将应用程序迁移到云端之前在本地尝试 Kubernetes。
minikube 是一个打包成虚拟机的本地 Kubernetes 集群。如果您使用的是 Windows、Linux 和 Mac，那就太好了，因为创建一个集群需要五分钟。
您还应该安装 kubectl，用于连接到您的集群的客户端。
您可以从[官方文档](https://kubernetes.io/docs/tasks/tools/)中找到有关如何安装 minikube 和 kubectl 的说明。

> 如果您在 Windows 上运行，您应该查看我们关于如何安装 Kubernetes 和 Docker 的详细指南。

您应该启动一个具有 8GB RAM 和一些额外配置的集群：

```
minikube start \
  --memory 8096 \
  --extra-config=controller-manager.horizontal-pod-autoscaler-upscale-delay=1m \
  --extra-config=controller-manager.horizontal-pod-autoscaler-downscale-delay=2m \
  --extra-config=controller-manager.horizontal-pod-autoscaler-sync-period=10s
```

> 请注意，如果您使用的是预先存在的 minikube 实例，您可以通过销毁它并重新创建它来调整 VM 的大小。 仅添加 --memory 8096 不会有任何效果。

验证安装是否成功。 
您应该会看到以表格形式列出的一些资源。 
集群已准备就绪，也许您现在应该开始部署？
其实，还不可以 😂。
你还是需要先准备一些东西。

部署到 Kubernetes 的应用程序必须打包为容器。 毕竟，Kubernetes 是一个容器编排器，所以它不能原生运行你的 jar。
容器类似于胖罐子：它们包含运行应用程序所需的所有依赖项。 甚至 JVM 也是容器的一部分。 所以从技术上讲，它们是一个更胖的胖罐子。
将应用程序打包为容器的流行技术是 Docker。

> 虽然是最受欢迎的，但 Docker 并不是唯一能够运行容器的技术。 其他流行的选项包括 rkt 和 lxd。

如果您没有安装 Docker，可以按照 [Docker 官方网站](https://docs.docker.com/install/)上的说明进行操作。
通常，您构建容器并将它们推送到注册表。 这类似于将 jar 发布到 Artifactory 或 Nexus。 但在这种特殊情况下，您将在本地工作并跳过注册表部分。 实际上，您将直接在 minikube 中创建容器镜像。
首先，按照此命令打印的说明将 Docker 客户端连接到 minikube：

```
minikube docker-env
```

> 请注意，如果您切换终端，您需要重新连接到 minikube 内部的 Docker 守护进程。 每次使用不同的终端时，都应遵循相同的说明。

并从项目的根目录构建容器镜像：

```
docker build -t spring-k8s-hpa .
```

您可以验证映像是否已构建并准备好运行：

```
docker images | grep spring
```

完美！
集群准备好了，你打包了你的应用程序，也许你现在已经准备好部署了？
是的，您终于可以要求 Kubernetes 部署应用程序了。

## 将您的应用程序部署到 Kubernetes

您的应用程序包含三个组件：
 * 呈现前端的 Spring Boot 应用程序
 * ActiveMQ 作为消息代理
 * 处理事务的 Spring Boot 后端
您应该分别部署这三个组件。
对于它们中的每一个，您应该创建：
 * 描述部署什么容器及其配置的部署对象
 * 一个服务对象，充当部署创建的应用程序的所有实例的负载均衡器
部署中应用程序的每个实例都称为 Pod。

{% asset_img 11.gif 示意图 width="400" %}

## 部署 ActiveMQ

让我们从 ActiveMQ 开始。
您应该创建一个包含以下内容的 activemq-deployment.yaml 文件：

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: queue
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: queue
    spec:
      containers:
      - name: web
        image: webcenter/activemq:5.14.3
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 61616
        resources:
          limits:
            memory: 512Mi
```

内容看着挺多的，但读起来还很简单：
 * 您从名为 webcenter/activemq 的官方注册表中请求了一个 activemq 容器
 * 容器在端口 61616 上公开消息代理
 * 为容器分配了 512MB 内存
 * 你要求一个副本——你的应用程序的一个实例
创建一个包含以下内容的 activemq-service.yaml 文件：

```
apiVersion: v1
kind: Service
metadata:
  name: queue
spec:
  ports:
  - port: 61616 
    targetPort: 61616
  selector:
    app: queue
```

幸运的是，这个模板更短！
yaml 内容如下：
 * 您创建了一个公开端口 61616 的负载均衡器
 * 传入流量被分发到所有具有 app: queue 类型标签的 Pod（参见上面的部署）
 * targetPort 是 Pod 暴露的端口
您可以使用以下方法创建资源：

```
kubectl create -f activemq-deployment.yaml
kubectl create -f activemq-service.yaml
```

您可以验证数据库的一个实例是否正在运行：

```
kubectl get pods -l=app=queue
```

## 部署前端

创建包含以下内容的 fe-deployment.yaml 文件：

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: spring-boot-hpa
        imagePullPolicy: IfNotPresent
        env:
        - name: ACTIVEMQ_BROKER_URL
          value: "tcp://queue:61616"
        - name: STORE_ENABLED
          value: "true"
        - name: WORKER_ENABLED
          value: "false"
        ports:
        - containerPort: 8080
        livenessProbe:
          initialDelaySeconds: 5
          periodSeconds: 5
          httpGet:
            path: /health
            port: 8080
        resources:
          limits:
            memory: 512Mi
```

部署看起来很像前一个。
不过，有一些新领域：
 * 有一个部分可以注入环境变量
 * 有一个 liveness probe 会告诉你应用程序何时准备好接受流量
创建一个 fe-service.yaml 文件，内容如下：

```
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  ports:
  - nodePort: 32000
    port: 80
    targetPort: 8080
  selector:
    app: frontend
  type: NodePort

```

您可以使用以下方法创建资源：

```
kubectl create -f fe-deployment.yaml
kubectl create -f fe-service.yaml
```

您可以验证前端应用程序的一个实例是否正在运行：

```
kubectl get pods -l=app=frontend
```

## 部署后端

创建一个包含以下内容的 backend-deployment.yaml 文件：

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: backend
      annotations:
        prometheus.io/scrape: 'true'
    spec:
      containers:
      - name: backend
        image: spring-boot-hpa
        imagePullPolicy: IfNotPresent
        env:
        - name: ACTIVEMQ_BROKER_URL
          value: "tcp://queue:61616"
        - name: STORE_ENABLED
          value: "false"
        - name: WORKER_ENABLED
          value: "true"
        ports:
        - containerPort: 8080
        livenessProbe:
          initialDelaySeconds: 5
          periodSeconds: 5
          httpGet:
            path: /health
            port: 8080
        resources:
          limits:
            memory: 512Mi
```

创建一个包含以下内容的 backend-service.yaml 文件：

```
apiVersion: v1
kind: Service
metadata:
  name: backend
  spec:
    ports:
    - nodePort: 31000
      port: 80
      targetPort: 8080
    selector:
      app: backend
    type: NodePort
```
您可以使用以下方法创建资源：

```
kubectl create -f backend-deployment.yaml
kubectl create -f backend-service.yaml
```

您可以验证后端的一个实例是否正在运行：

```
kubectl get pods -l=app=backend
```
部署完成。

**不过，它真的已经开始工作了吗？**

您可以使用以下命令在浏览器中访问该应用程序：

```
minikube service backend
minikube service frontend
```

如果有效，您应该尝试购买一些物品！
worker是否在处理事务？
是的，如果有足够的时间，worker 将处理所有待处理的消息。
Congratulations!
您刚刚将应用程序部署到 Kubernetes！

## 手动扩展以满足不断增长的需求

单个woker可能无法处理大量消息。 事实上，它一次只能处理一条消息。
如果您决定购买数千件商品，则需要数小时才能清除队列。
此时，您有两个选择：
 * 您可以手动放大和缩小
 * 您可以创建自动缩放规则以自动放大或缩小
让我们先从基础开始。
您可以通过以下方式将后端扩展到三个实例：

```
kubectl scale --replicas=5 deployment/backend
```

您可以验证 Kubernetes 是否创建了另外五个实例：

```
kubectl get pods
```

该应用程序可以处理五倍以上的消息。
一旦worker排空队列，您可以通过以下方式缩减：

```
kubectl scale --replicas=1 deployment/backend
```

如果您知道什么时候访问您的服务的流量最多，通过手动方式向上和向下扩展会是一种非常棒的体验。
如果您不这样做，设置自动缩放器允许应用程序自动缩放而无需人工干预。
您只需要定义一些规则。

### 公开应用程序指标

Kubernetes 如何知道何时扩展您的应用程序？
很简单，你必须告诉它。
自动缩放器通过监控指标来工作。 只有这样它才能增加或减少您的应用程序的实例。
因此，您可以将队列的长度作为指标公开，并要求自动缩放器查看该值。 队列中的待处理消息越多，Kubernetes 将创建的应用程序实例就越多。

**那么如何公开这些指标呢？**

应用程序有一个 /metrics 端点来公开队列中的消息数。 如果您尝试访问该页面，您会注意到以下内容：

```
# HELP messages Number of messages in the queue
# TYPE messages gauge
messages 0
```

应用程序不会将指标公开为 JSON 格式。 格式是纯文本，是公开 [Prometheus 指标](https://prometheus.io/docs/concepts/metric_types/)的标准。 不要担心记住格式。 大多数时候，您将使用 Prometheus 客户端库之一。

### 在 Kubernetes 中使用应用程序指标

您几乎已准备好进行自动缩放——但您应该先安装指标服务器。 事实上，Kubernetes 默认不会从您的应用程序中提取指标。 如果您愿意，您应该启用自定义指标 API。
要安装自定义指标 API，您还需要 Prometheus — 一个时间序列数据库。 安装 Custom Metrics API 所需的所有文件都方便地打包在 learnk8s/spring-boot-k8s-hpa 中。
您应该下载该存储库的内容并将当前目录更改为该项目的监视文件夹中。

```
cd spring-boot-k8s-hpa/monitoring
```

您可以从那里创建自定义指标 API：

```
kubectl create -f ./metrics-server
kubectl create -f ./namespaces.yaml
kubectl create -f ./prometheus
kubectl create -f ./custom-metrics-api
```

您应该等到以下命令返回自定义指标列表：

```
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .
```

任务完成！

### 您已准备好使用指标。

事实上，您应该已经找到了队列中消息数量的自定义指标：

```
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/messages" | jq .
```

恭喜，您有一个公开指标的应用程序和一个使用它们的指标服务器。
您终于可以启用自动缩放器了！

### Kubernetes 中的自动扩展部署

Kubernetes 有一个名为 Horizontal Pod Autoscaler 的对象，用于监控部署并向上和向下扩展 Pod 的数量。
您将需要其中之一来自动扩展您的实例。
您应该创建一个包含以下内容的 hpa.yaml 文件：

```
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: spring-boot-hpa
spec:
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: backend 
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metricName: messages
      targetAverageValue: 10
```

该文件很神秘，所以让我为您翻译一下：
Kubernetes 监视 scaleTargetRef 中指定的部署。 在这种情况下，它是worker。
您正在使用消息指标来扩展您的 Pod。 当队列中的消息超过 10 条时，Kubernetes 将触发自动缩放。
至少，部署应该有两个 Pod。 十个 Pod 是上限。
您可以使用以下方法创建资源：

```
kubectl create -f hpa.yaml
```

提交自动缩放器后，您应该注意到后端的副本数为两个。 这是有道理的，因为您要求自动缩放器始终至少运行两个副本。
您可以检查触发自动缩放器的条件以及由此生成的事件：

```
kubectl describe hpa
```

自动扩缩器表明它能够将 Pod 扩展到 2 个，并准备好监控部署。

**令人兴奋的东西，但它有效吗？**

### 负载测试

只有一种方法可以知道它是否有效：在队列中创建大量消息。
转到前端应用程序并开始添加大量消息。 添加消息时，请使用以下方法监控 Horizontal Pod Autoscaler 的状态：

```
kubectl describe hpa
```

Pod 的数量从 2 个增加到 4 个，然后是 8 个，最后是 10 个。
**该应用程序随消息数量而扩展！ 欢呼吧！**
您刚刚部署了一个完全可扩展的应用程序，该应用程序可根据队列中待处理消息的数量进行扩展。
附带说明一下，缩放算法如下：

```
MAX(CURRENT_REPLICAS_LENGTH * 2, 4)
```

此外，每一次放大都会每分钟重新评估一次，而每两分钟缩小一次。
以上都是可以调整的设置。
不过，你还没有完成。

### 有什么比自动缩放实例更好的呢？ 自动缩放集群。

跨节点扩展 Pod 的效果非常好。 但是，如果集群中没有足够的容量来扩展 Pod 怎么办？
如果达到峰值容量，Kubernetes 将使 Pod 处于挂起状态并等待更多资源可用。
如果您可以使用类似于 Horizontal Pod Autoscaler 但用于节点的自动缩放器，那就太好了。

您可以拥有一个集群自动扩缩器，在您需要更多资源时向 Kubernetes 集群添加更多节点。

{% asset_img 12.gif 示意图 width="400" %}

集群自动缩放器有不同的形状和大小。 它也是特定于云提供商的。

### 回顾一下
大规模设计应用程序需要仔细规划和测试。
基于队列的架构是一种出色的设计模式，可以解耦微服务并确保它们可以独立扩展和部署。
虽然您可以推出部署脚本，但利用 Kubernetes 等容器编排器来自动部署和扩展应用程序会更容易。