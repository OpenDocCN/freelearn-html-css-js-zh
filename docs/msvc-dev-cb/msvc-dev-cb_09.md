# 第九章：微服务部署

在本章中，我们将介绍以下菜谱：

+   配置您的服务以在容器中运行

+   使用 Docker Compose 运行多容器应用程序

+   在 Kubernetes 上部署您的服务

+   使用金丝雀部署进行测试发布

# 简介

多年来，我们向用户交付软件的方式发生了巨大的变化。在不太遥远的过去，通过在服务器集合上运行 shell 脚本来部署到生产环境是很常见的，这些服务器从某种源代码存储库中拉取更新。这种方法的问题很明显——扩展这个方法很困难，启动服务器容易出错，部署可能会陷入不希望的状态，从而导致用户体验不可预测。

配置管理系统的出现，如**Chef**或**Puppet**，在某种程度上改善了这种情况。与在远程服务器上运行自定义 bash 脚本或命令不同，远程服务器可以被标记为一种角色，指示它们如何配置和安装软件。声明式自动化配置的风格更适合大规模软件部署。还采用了服务器自动化工具，如**Fabric**或**Capistrano**；它们旨在自动化将代码推送到生产的过程，并且至今仍非常受欢迎，用于不在容器中运行的应用程序。

容器已经彻底改变了我们交付软件的方式。容器允许开发者将他们的代码及其所有依赖项打包，包括库、运行时、操作系统工具和配置。这使得代码的交付无需配置主机服务器，通过减少移动部件的数量，大大简化了过程。

在容器中运输服务的过程被称为**不可变基础设施**，因为一旦构建了镜像，通常不会对其进行更改；相反，服务的新版本会导致构建新的镜像。

软件部署方式中的另一个重大变化是十二要素方法的普及([`12factor.net/`](https://12factor.net/))。**十二要素**（或**12f**，如通常书写）是由 Heroku 的工程师编写的一系列指南。在核心上，十二要素应用程序旨在与环境松散耦合，从而产生可以与各种日志工具、配置系统、软件包管理系统和源代码控制系统一起使用的服务。可以说，十二要素应用程序最普遍采用的概念是配置通过环境变量访问，日志输出到标准输出。正如我们在前面的章节中看到的，这是我们与 Vault 等系统集成的这种方式。这些章节值得一读，但我们在本书中已经遵循了许多十二要素中描述的概念。

在本章中，我们将讨论容器、编排和调度，以及将更改安全地发送给用户的各种方法。这是一个非常活跃的话题，新的技术正在被发明和讨论，但本章中的菜谱应该是一个很好的起点，特别是如果你习惯于在虚拟机或裸机服务器上部署单体应用。

# 配置你的服务在容器中运行

正如我们所知，服务由源代码和配置组成。例如，用 Java 编写的服务可以被打包成一个包含编译后的 Java 字节码的 **Java 存档**（**JAR**）文件，以及配置和属性文件等资源。一旦打包，JAR 文件就可以在任何运行 **Java 虚拟机**（**JVM**）的机器上执行。

然而，为了使这一切工作，我们想要运行服务的机器必须安装了 JVM。通常，它必须是 JVM 的特定版本。此外，机器可能需要安装一些其他实用程序，或者可能需要访问共享文件系统。虽然这些不是服务本身的组成部分，但它们构成了我们所说的服务的运行时环境。

Linux 容器是一种技术，允许开发者将应用程序或服务及其完整的运行时环境打包在一起。容器将特定应用程序的运行时与容器运行的宿主机的运行时分离出来。

这使得应用程序更加可移植，使得将服务从一个环境迁移到另一个环境变得更加容易。工程师可以在他们的笔记本电脑上运行一个服务，然后将其移动到预生产环境，然后进入生产环境，而无需更改容器本身。容器还允许你轻松地在同一台机器上运行多个服务，因此提供了更多灵活性，以部署应用程序架构，并提供了优化运营成本的机会。

Docker 是一个容器运行时和一系列工具，允许你为你的服务创建自包含的执行环境。今天还有其他流行的容器运行时被广泛使用，但 Docker 被设计成使容器可移植和灵活，使其成为构建服务容器的理想选择。

在这个菜谱中，我们将使用 Docker 创建一个打包我们的消息服务的镜像。我们将通过创建一个 `Dockerfile` 文件，并使用 Docker 命令行工具来创建镜像，然后运行该镜像作为容器来实现这一点。

# 如何操作…

这个菜谱的步骤如下：

1.  首先，从上一章打开我们的消息服务项目。在项目的根目录下创建一个名为 `Dockerfile` 的新文件：

```js
FROM openjdk:8-jdk-alpine
VOLUME /tmp
EXPOSE 8082
ARG JAR_FILE=build/libs/message-service-1.0-SNAPSHOT.jar
ADD ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

1.  `Dockerfile` 文件定义了我们将用于构建 message-service 镜像的基础镜像。在这种情况下，我们基于一个带有 OpenJDK 8 的 Alpine Linux 镜像。接下来，我们暴露了我们的服务绑定的端口，并定义了如何将我们的服务作为 JAR 文件打包后运行。我们现在可以使用 `Dockerfile` 文件来构建镜像。这可以通过以下命令完成：

```js
$ docker build . -t message-service:0.1.1
```

1.  你可以通过运行 `docker images` 并查看我们的镜像是否列出来验证前面的命令是否成功。现在我们准备好通过在容器中执行我们的服务来运行消息服务。这是通过 `docker run` 命令完成的。我们还将给它一个端口映射并指定我们想要用于运行服务的镜像：

```js
$ docker run -p 0.0.0.0:8082:8082 message-service:0.1.1
```

# 使用 Docker Compose 运行多容器应用

服务很少独立运行。一个微服务通常连接到某种类型的数据存储，并且可能有其他运行时依赖。为了在微服务上工作，有必要在开发者的机器上本地运行它。要求工程师手动安装和管理服务的所有运行时依赖以在微服务上工作将是不切实际且耗时的。相反，我们需要一种自动管理运行时服务依赖的方法。

容器通过将运行时环境和配置与应用程序代码一起打包为可运输的工件，使得服务更加便携。为了最大限度地发挥使用容器进行本地开发的益处，能够声明所有依赖并在单独的容器中运行它们将是非常好的。这正是 Docker Compose 设计来做的。

Docker Compose 使用声明性的 YAML 配置文件来确定应用程序应该如何在多个容器中执行。这使得快速启动服务、数据库以及服务的任何其他运行时依赖变得非常容易。

在这个食谱中，我们将遵循前一个食谱中的某些步骤来为 authentication-service 项目创建一个 `Dockerfile` 文件。然后，我们将创建一个 Docker Compose 文件，指定 MySQL 作为 authentication-service 的依赖项。然后，我们将查看如何配置我们的项目并在本地运行它，一个容器运行我们的应用程序，另一个运行数据库服务器。

# 如何操作...

对于这个食谱，你需要执行以下步骤：

1.  打开 authentication-service 项目并创建一个名为 `Dockerfile` 的新文件：

```js
FROM openjdk:8-jdk-alpine
VOLUME /tmp
EXPOSE 8082
ARG JAR_FILE=build/libs/authentication-service-1.0-SNAPSHOT.jar
ADD ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

1.  Docker Compose 使用一个名为 `docker-compose.yml` 的文件来声明容器化应用应该如何运行：

```js
version: '3'
services:
 authentication:
 build: .
 ports:
 - "8081:8081"
 links:
 - docker-mysql
 environment:
 DATABASE_HOST: 'docker-mysql'
 DATABASE_USER: 'root'
 DATABASE_PASSWORD: 'root'
 DATABASE_NAME: 'user_credentials'
 DATABASE_PORT: 3306
 docker-mysql:
 ports:
 - "3306:3306"
 image: mysql
 restart: always
 environment:
 MYSQL_ROOT_PASSWORD: 'root'
 MYSQL_DATABASE: 'user_credentials'
 MYSQL_ROOT_HOST: '%'
```

1.  由于我们将连接到在 `docker-mysql` 容器中运行的 MySQL 服务器，我们需要修改 authentication-service 的配置，以便在连接到 MySQL 时使用该主机：

```js
server:
 port: 8081

spring:
 jpa.hibernate.ddl-auto: create
 datasource.url: jdbc:mysql://docker-mysql:3306/user_credentials
 datasource.username: root
 datasource.password: root

hibernate.dialect: org.hibernate.dialect.MySQLInnoDBDialect

secretKey: supers3cr3t
```

1.  你现在可以使用以下命令运行 authentication-service 和 MySQL：

```js
$ docker-compose up
Starting authentication-service_docker-mysql_1 ...
```

1.  就这样！authentication-service 应该现在已经在本地容器中运行了。

# 在 Kubernetes 上部署你的服务

容器通过允许你将代码、依赖项和运行时环境打包在一个工件中，使得服务可移植。部署容器通常比部署不运行在容器中的应用程序更容易。主机不需要任何特殊的配置或状态；它只需要能够执行容器运行时。在单个主机上部署一个或多个容器的能力，在管理生产环境时产生了另一个挑战——调度和编排容器在特定主机上运行并管理扩展。

Kubernetes 是一个开源的容器编排工具。它负责调度、管理和扩展你的容器化应用程序。使用 Kubernetes，你无需担心将容器部署到一台或多台特定的主机上。相反，你只需声明你的容器需要哪些资源，然后让 Kubernetes 决定如何执行工作（容器运行在哪个主机上，它旁边运行哪些服务等等）。Kubernetes 是从 Google 工程师发布的 **Borg 论文** ([`research.google.com/pubs/pub43438.html`](https://research.google.com/pubs/pub43438.html)) 中发展而来的，这篇论文描述了他们如何使用 Borg 集群管理器在 Google 的数据中心管理服务。

Kubernetes 是 Google 在 2014 年启动的一个开源项目，并且被部署容器代码的组织广泛采用。

安装和管理 Kubernetes 集群超出了本书的范围。幸运的是，一个名为 **Minikube** 的项目允许你在开发机器上轻松运行单个节点的 Kubernetes 集群。尽管集群只有一个节点，但当你部署服务时与 Kubernetes 交互的方式通常是一样的，所以这里的步骤可以适用于任何 Kubernetes 集群。

在这个菜谱中，我们将安装 Minikube，启动单个节点的 Kubernetes 集群，并部署我们在前几章中使用的 `message-service` 命令。我们将使用 Kubernetes CLI 工具 (`kubectl`) 与 Minikube 进行交互。

# 如何操作...

对于这个菜谱，你需要完成以下步骤：

1.  为了演示如何将我们的服务部署到 `kubernetes` 集群，我们将使用一个名为 `minikube` 的工具。`minikube` 工具使得在可以运行在笔记本电脑或开发机器上的虚拟机（VM）上运行单个节点的 `kubernetes` 集群变得容易。在 macOS X 上，你可以使用 HomeBrew 来安装它：

```js
$ brew install minikube
```

1.  在这个菜谱中，我们还将使用 `kubernetes` CLI 工具，所以请安装它们。在 macOS X 上，使用 HomeBrew，你可以输入以下内容：

```js
$ brew install kubernetes-cli
```

1.  现在我们已经准备好启动我们的单个节点 `kubernetes` 集群。你可以通过运行 `minikube start` 来完成这个操作：

```js
$ minikube start
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file
```

1.  接下来，将 `minikube` 集群设置为 `kubectl` CLI 工具的默认配置：

```js
$ kubectl config use-context minikube
Switched to context "minikube".
```

1.  通过运行 `cluster-info` 命令来验证一切配置是否正确：

```js
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

为了进一步调试和诊断集群问题，使用 `kubectl cluster-info dump`。

1.  现在，你应该能够在浏览器中启动`kubernetes`仪表板：

```js
$ minikube dashboard
Waiting, endpoint for service is not ready yet...
Opening kubernetes dashboard in default browser...
```

1.  `minikube`工具使用多个环境变量来配置 CLI 客户端。使用以下命令评估环境变量：

```js
$ eval $(minikube docker-env)
```

1.  接下来，我们将使用之前配方中创建的`Dockerfile`文件为我们的服务构建 docker 镜像：

```js
$ docker build -t message-service:0.1.1
```

1.  最后，在`kubernetes`集群上运行`message-service`命令，告诉`kubectl`正确的镜像和要暴露的端口：

```js
$ kubectl run message-service --image=message-service:0.1.1 --port=8082 --image-pull-policy=Never
```

1.  我们可以通过列出集群上的 pods 来验证`message-service`命令是否在`kubernetes`集群中运行：

```js
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
message-service-87d85dd58-svzmj 1/1 Running 0 3s
```

1.  为了访问`message-service`命令，我们需要将其作为新的服务暴露出来：

```js
$ kubectl expose deployment message-service --type=LoadBalancer
service/message-service exposed
```

1.  我们可以通过列出`kubernetes`服务上的服务来验证上一条命令：

```js
$ kubectl get services

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 59d
message-service LoadBalancer 10.105.73.177 <pending> 8082:30382/TCP 4s
```

1.  `minikube`工具有一个方便的命令来访问在`kubernetes`集群上运行的服务。运行以下命令将列出`message-service`命令正在运行的 URL：

```js
$ minikube service list message-service
|-------------|----------------------|-----------------------------|
| NAMESPACE | NAME | URL |
|-------------|----------------------|-----------------------------|
| default | kubernetes | No node port |
| default | message-service | http://192.168.99.100:30382 |
| kube-system | kube-dns | No node port |
| kube-system | kubernetes-dashboard | http://192.168.99.100:30000 |
|-------------|----------------------|-----------------------------|
```

1.  使用`curl`尝试向服务发送请求以验证其是否正常工作。恭喜！你已经在`kubernetes`上部署了`message-service`命令。

# 使用金丝雀部署进行测试发布

部署最佳实践的改进极大地提高了多年来部署的稳定性。自动化可重复步骤、标准化我们的应用程序与运行时环境的交互方式，以及将我们的应用程序代码与运行时环境打包在一起，都使得部署比以前更安全、更容易。

将新代码引入生产环境并非没有风险。本章中讨论的所有技术都有助于防止可预测的错误，但它们并不能阻止实际软件错误对我们所编写的软件用户产生负面影响。金丝雀部署是一种减少这种风险并增加对部署到生产环境的新代码的信心的一种技术。

在金丝雀部署中，你首先将代码发送到一小部分生产流量。然后你可以监控指标、日志、跟踪或其他允许你观察软件工作情况的工具。一旦你确信一切按预期进行，你可以逐渐增加接收更新版本流量的百分比，直到所有生产流量都由你服务的最新版本提供服务。

“金丝雀部署”这个术语来自煤矿工人用来保护自己免受一氧化碳或甲烷中毒的技术。通过在矿井中放置金丝雀，有毒气体会在矿工之前杀死金丝雀，给矿工一个提前的警告信号，告诉他们应该离开。同样，金丝雀部署允许我们向一小部分用户暴露风险，而不会影响生产环境的其余部分。幸运的是，在将代码部署到生产环境时，不需要伤害任何动物。

金丝雀部署过去一直很难正确实施。以这种方式运输软件的团队通常必须想出某种功能切换解决方案，以控制对正在部署的应用程序特定版本的请求。幸运的是，容器使这变得容易得多，而 Kubernetes 则使其变得更加容易。

在这个菜谱中，我们将使用金丝雀部署来部署我们的`message-service`应用程序的更新。由于 Kubernetes 能够从 Docker 容器仓库拉取镜像，我们将在本地运行一个仓库。通常，你会使用自托管的仓库或像*Docker Hub*或*Google Container Registry*这样的服务。首先，我们将确保在`minikube`中运行一个稳定的`message-service`命令版本，然后我们将引入一个更新，并逐渐将其推出到 100%流量。

# 如何做到这一点...

按照以下步骤设置金丝雀部署：

1.  打开我们在之前菜谱中工作的`message-service`项目。将以下`Dockerfile`文件添加到项目的根目录：

```js
FROM openjdk:8-jdk-alpine
VOLUME /tmp
EXPOSE 8082
ARG JAR_FILE=build/libs/message-service-1.0-SNAPSHOT.jar
ADD ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

1.  为了让 Kubernetes 知道服务是否正在运行，我们需要添加一个存活性探针端点。打开`MessageController.java`文件，并添加一个方法来响应`/ping`路径的 GET 请求：

```js
package com.packtpub.microservices.ch09.message.controllers;

import com.packtpub.microservices.ch09.message.MessageRepository;
import com.packtpub.microservices.ch09.message.clients.SocialGraphClient;
import com.packtpub.microservices.ch09.message.exceptions.MessageNotFoundException;
import com.packtpub.microservices.ch09.message.exceptions.MessageSendForbiddenException;
import com.packtpub.microservices.ch09.message.models.Message;
import com.packtpub.microservices.ch09.message.models.UserFriendships;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.scheduling.annotation.Async;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.servlet.support.ServletUriComponentsBuilder;

import java.net.URI;
import java.util.List;
import java.util.concurrent.CompletableFuture;

@RestController
public class MessageController {

    @Autowired
    private MessageRepository messagesStore;

    @Autowired
    private SocialGraphClient socialGraphClient;

    @RequestMapping(path = "/{id}", method = RequestMethod.GET, produces = "application/json")
    public Message get(@PathVariable("id") String id) throws MessageNotFoundException {
        return messagesStore.get(id);
    }

    @RequestMapping(path = "/ping", method = RequestMethod.GET)
    public String readinessProbe() {
        return "ok";
    }

    @RequestMapping(path = "/", method = RequestMethod.POST, produces = "application/json")
    public ResponseEntity<Message> send(@RequestBody Message message) throws MessageSendForbiddenException {
        List<String> friendships = socialGraphClient.getFriendships(message.getSender());

        if (!friendships.contains(message.getRecipient())) {
            throw new MessageSendForbiddenException("Must be friends to send message");
        }

        Message saved = messagesStore.save(message);
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest().path("/{id}")
                .buildAndExpand(saved.getId()).toUri();
        return ResponseEntity.created(location).build();
    }

    @RequestMapping(path = "/user/{userId}", method = RequestMethod.GET, produces = "application/json")
    public ResponseEntity<List<Message>> getByUser(@PathVariable("userId") String userId) throws MessageNotFoundException  {
        List<Message> inbox = messagesStore.getByUser(userId);
        if (inbox.isEmpty()) {
            throw new MessageNotFoundException("No messages found for user: " + userId);
        }
        return ResponseEntity.ok(inbox);
    }

    @Async
    public CompletableFuture<Boolean> isFollowing(String fromUser, String toUser) {
        String url = String.format(
                "http://localhost:4567/followings?user=%s&filter=%s",
                fromUser, toUser);

        RestTemplate template = new RestTemplate();
        UserFriendships followings = template.getForObject(url, UserFriendships.class);

        return CompletableFuture.completedFuture(
                followings.getFriendships().isEmpty()
        );
    }
}
```

1.  让我们在 5000 端口上启动我们的容器仓库：

```js
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

1.  由于我们正在使用一个未配置有效 SSL 证书的本地仓库，请以能够从非安全仓库拉取的能力启动`minikube`：

```js
$ minikube start --insecure-registry 127.0.0.1
```

1.  构建名为`message-service`的 docker 镜像，然后使用以下命令将镜像推送到本地容器仓库：

```js
$ docker build . -t message-service:0.1.1
...
$ docker tag message-service:0.1.1 localhost:5000/message-service
...
$ docker push localhost:5000/message-service
```

1.  **Kubernetes Deployment**对象描述了 Pod 和`ReplicaSet`的期望状态。在我们的部署中，我们将指定我们希望始终运行三个`message-service` Pod 的副本，并且我们将指定我们之前创建的存活性探针。为了为我们的`message-service`创建一个部署，创建一个名为`deployment.yaml`的文件，并包含以下内容：

```js
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: message-service
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: "message-service"
        track: "stable"
    spec:
      containers:
        - name: "message-service"
          image: "localhost:5000/message-service"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8082
          livenessProbe:
            httpGet:
              path: /ping
              port: 8082
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 30
            timeoutSeconds: 1
```

1.  接下来，使用`kubectl`，我们将创建我们的部署对象：

```js
$ kubectl create -f deployment.yaml
```

1.  你现在可以通过运行`kubectl get pods`来验证我们的部署是活跃的，并且 Kubernetes 正在创建 Pod 副本：

```js
$ kubectl get pods
```

1.  现在我们的应用程序正在 Kubernetes 中运行，下一步是创建一个更新并将其推出到 Pod 子集。首先，我们需要创建一个新的 docker 镜像；在这种情况下，我们将称之为版本 0.1.2，并将其推送到本地仓库：

```js
$ docker build . -t message-service:0.1.2
...
$ docker tag message-service:0.1.2 localhost:5000/message-service
$ docker push localhost:5000/message-service
```

1.  现在我们可以配置一个部署，在将其推出到其他 Pod 之前先运行我们的最新版本镜像。
