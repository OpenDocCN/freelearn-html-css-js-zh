# 扩展

在本章中，我们将涵盖以下菜谱：

+   使用 Vegeta 进行微服务负载测试

+   使用 Gatling 进行微服务负载测试

+   构建自动扩展集群

# 简介

使用微服务而不是单体架构的一个显著优势是，微服务可以单独扩展以满足它们所服务的独特流量需求。必须为每个请求执行工作的服务将具有与只需要为特定类型的请求执行工作的服务非常不同的扩展需求。

由于微服务封装了对单个领域实体的所有权，它们可以独立进行负载测试。它们还可以根据需求自动配置扩展。在本章中，我们将讨论使用两种不同的负载测试工具进行负载测试，并在 AWS 中设置自动扩展组，以便按需扩展。最后，我们将讨论容量规划策略。

# 使用 Vegeta 进行微服务负载测试

负载测试是预测您的服务随时间如何表现的重要部分。当我们进行负载测试时，我们不应该只问简单的问题，例如“*我们的系统每秒能处理多少请求？*”相反，我们应该试图了解我们的整个系统在各种负载条件下的表现。为了回答这个问题，我们需要了解构成我们系统的基础设施以及特定服务所依赖的依赖关系。

例如，该服务是否位于负载均衡器后面？CDN 呢？还使用了哪些其他缓存机制？所有这些问题以及更多都可以通过我们系统良好的可观察性得到解答。

**Vegeta**是一个开源的负载测试工具，旨在以恒定的请求速率测试 HTTP 服务。它是一个多功能的工具，可以用作命令行工具或库。在这个菜谱中，我们将专注于使用命令行工具。Vegeta 允许你将目标指定为单独文件中的 URL，可选地带有自定义头和请求体，这些可以作为命令行工具的输入。然后，命令行工具可以攻击文件中的目标，有各种选项来控制请求速率和持续时间，以及其他变量。

在这个菜谱中，我们将使用 Vegeta 测试我们在前几章中一直在使用的消息服务。我们将测试一个简单的请求路径，包括创建新消息和检索消息列表。

# 如何实现...

让我们看看以下步骤：

1.  我们将修改我们的消息服务并添加一个新的端点，允许我们查询特定用户的全部消息。这引入了收件箱的概念，因此我们将修改我们的`MessageRepository`类以添加一个新的内存映射，将用户名映射到消息列表，如下面的代码所示。请注意，在生产系统中，我们会选择一个更持久和灵活的存储方案，但这对演示目的已经足够了：

```js
package com.packtpub.microservices.ch08.message;

import com.packtpub.microservices.ch08.message.exceptions.MessageNotFoundException;
import com.packtpub.microservices.ch08.message.models.Message;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class MessageRepository {

    private ConcurrentHashMap<String, Message> messages;
    private ConcurrentHashMap<String, List<Message>> inbox;

    public MessageRepository() {
        messages = new ConcurrentHashMap<>();
        inbox = new ConcurrentHashMap<>();
    }

    public Message save(Message message) {
        UUID uuid = UUID.randomUUID();
        Message saved = new Message(uuid.toString(), message.getSender(), message.getRecipient(),
                message.getBody(), message.getAttachmentUri());
        messages.put(uuid.toString(), saved);
        List<Message> userInbox = inbox.getOrDefault(message.getRecipient(), new ArrayList<>());
        userInbox.add(saved);
        inbox.put(message.getRecipient(), userInbox);
        return saved;
    }

    public Message get(String id) throws MessageNotFoundException {
        if (messages.containsKey(id)) {
            return messages.get(id);
        } else {
            throw new MessageNotFoundException("Message " + id + " could not be found");
        }
    }

    public List<Message> getByUser(String userId) {
        return inbox.getOrDefault(userId, new ArrayList<>());
    }
}
```

1.  修改`MessageController`以添加端点本身：

```js
package com.packtpub.microservices.ch08.message.controllers;

import com.packtpub.microservices.ch08.message.MessageRepository;
import com.packtpub.microservices.ch08.message.clients.SocialGraphClient;
import com.packtpub.microservices.ch08.message.exceptions.MessageNotFoundException;
import com.packtpub.microservices.ch08.message.exceptions.MessageSendForbiddenException;
import com.packtpub.microservices.ch08.message.exceptions.MessagesNotFoundException;
import com.packtpub.microservices.ch08.message.models.Message;
import com.packtpub.microservices.ch08.message.models.UserFriendships;
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
    public ResponseEntity<List<Message>> getByUser(@PathVariable("userId") String userId) throws MessageNotFoundException {
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

1.  我们需要一个模拟的社交图服务，因此在一个名为`socialgraph.rb`的文件中创建以下 Ruby 脚本并运行它：

```js
require 'sinatra'

get '/friendships/:user' do
    content_type :json
    {
        username: "user:32134",
        friendships: [
            "user:12345"
        ]
    }.to_json
end
```

1.  安装`vegeta`。如果你在 Mac OS X 上并且已经安装了 HomeBrew，你可以直接使用以下命令：

```js
$ brew update && brew install vegeta
```

1.  在我们能够使用`vegeta`启动附件之前，我们需要创建一个`targets`文件。我们将发出的第一个请求将创建一个带有指定请求体的消息。第二个请求将通过用户 ID 获取消息列表。创建一个名为`message-request-body.json`的文件，如下面的代码所示：

```js
{
    "sender": "user:32134",
    "recipient": "user:12345",
    "body": "Hello there!",
    "attachment_uri": "http://foo.com/image.png"
}
```

1.  创建另一个名为`targets.txt`的文件，如下面的代码所示：

```js
POST http://localhost:8082/
Content-Type: application/json
@message-request-body.json

GET http://localhost:8082/user:12345
```

1.  我们的消息服务和模拟社交图服务都已运行，我们现在可以使用以下代码来对这些两个服务进行负载测试：

```js
$ cat targets.txt| vegeta attack -duration=60s -rate=100 | vegeta report -reporter=text

Requests      [total, rate]            6000, 100.01
Duration      [total, attack, wait]    1m0.004668981s, 59.99172349s, 12.945491ms
Latencies     [mean, 50, 95, 99, max]  10.683968ms, 5.598656ms, 35.108562ms, 98.290388ms, 425.186942ms
Bytes In      [total, mean]            667057195, 111176.20
Bytes Out     [total, mean]            420000, 70.00
Success       [ratio]                  99.80%
Status Codes  [code:count]             201:3000  500:12  200:2988
Error Set:
50
```

尝试不同的持续时间值和请求速率，看看系统行为如何变化。如果你将速率增加到 1,000，会发生什么？根据硬件和其他因素，单线程的 Ruby 模拟服务可能会超负荷运行并触发我们添加到消息服务的熔断器。这可能会改变某些细节，例如成功率，因此这是一个重要的观察点。如果你单独对模拟 Ruby 服务进行负载测试，会发生什么？

在这个菜谱中，我们负载测试了依赖于社交图服务的消息服务。这两个服务都在本地运行，这对于演示目的来说是必要的，并让我们对这两个系统的行为有了一些了解。在生产系统中，在生产环境中对服务进行负载测试至关重要，以确保你包括了所有参与服务请求的基础设施（负载均衡器、缓存等）。在生产系统中，你还可以监控仪表板，寻找系统在负载条件下的行为变化。

# 使用 Gatling 对微服务进行负载测试

Gatling 是一个开源的负载测试工具，它允许用户使用基于*Scala 的 DSL*脚本来编写自定义场景。场景可以超越简单的直线路径测试，包括多个步骤，甚至可以模拟用户行为，如暂停并根据测试输出做出如何继续的决定。Gatling 可以用来自动化微服务或基于浏览器的 Web 应用的负载测试。

在上一个菜谱中，我们使用 Vegeta 向我们的消息服务发送恒定的请求速率。我们的请求路径创建了一条新消息，然后检索了用户的全部消息。这种方法的优势在于能够测试随着消息列表的增长，检索用户所有消息的响应时间。Vegeta 擅长此类测试，但由于它从静态文件中获取攻击目标，因此你不能使用 Vegeta 根据先前请求的响应构建动态请求路径。

由于 Gatling 使用 DSL 来编写负载测试场景，因此可以发出请求，捕获响应的一些元素，并使用该输出来做出关于未来请求的决定。在本食谱中，我们将使用 Gatling 编写一个涉及创建消息然后通过其 ID 检索该特定消息的负载测试场景。这与之前食谱中做的测试非常不同，因此这是一个很好的机会来展示 Vegeta 和 Gatling 之间的差异。

# 如何做到这一点...

让我们检查以下步骤：

1.  为你的平台下载`gatling`。Gatling 以 ZIP 包的形式分发，可在[`gatling.io/download/`](https://gatling.io/download/)下载。将包解压到你的选择目录中：

```js
$ unzip gatling-charts-highcharts-bundle-2.3.1-bundle.zip
...
$ cd gatling-charts-highcharts-bundle-2.3.1
```

1.  `gatling`的模拟默认放置在`user-files/simulations`目录中。创建一个名为`messageservice`的新子目录和一个名为`BasicSimulation.scala`的新文件。这是包含描述你的场景的代码的文件。在我们的场景中，我们将使用 Gatling DSL 来编写一个 POST 请求到创建消息端点，然后是一个 GET 请求到消息端点，如下面的代码所示：

```js
package messageservice

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class BasicSimulation extends Simulation {

  val httpConf = http
    .baseURL("http://localhost:8082")
    .acceptHeader("application/json")

  val scn = scenario("Create a message")
    .exec(
      http("createMessage")
        .post("/")
        .header("Content-Type", "application/json")
        .body(StringBody("""{"sender": "user:32134", "recipient": "user:12345", "body": "Hello there!", "attachment_uri": "http://foo.com/image.png"}"""))
        .check(header(HttpHeaderNames.Location).saveAs("location"))

    )
    .pause(1)
    .exec(
      http("getMessage")
        .get("${location}")
    )

  setUp(scn.inject(atOnceUsers(50)).protocols(httpConf))
}

```

1.  创建与之前食谱中使用的相同的模拟 Ruby 服务并运行它：

```js
require 'sinatra'

get '/friendships/:user' do
    content_type :json
    {
        username: "user:32134",
        friendships: [
            "user:12345"
        ]
    }.to_json
end
```

1.  同时运行 Ruby 模拟服务和我们的消息服务。从 Gatling 目录中，通过运行`bin/gatling.sh`启动 Gatling。你将被提示选择要运行的模拟。选择`messageservice.BasicSimulation`：

```js
$ bin/gatling.sh
GATLING_HOME is set to /Users/posman/projects/microservices-cookbook/chapter08/gatling-charts-highcharts-bundle-2.3.1
Choose a simulation number:
 [0] computerdatabase.BasicSimulation
 [1] computerdatabase.advanced.AdvancedSimulationStep01
 [2] computerdatabase.advanced.AdvancedSimulationStep02
 [3] computerdatabase.advanced.AdvancedSimulationStep03
 [4] computerdatabase.advanced.AdvancedSimulationStep04
 [5] computerdatabase.advanced.AdvancedSimulationStep05
 [6] messageservice.BasicSimulation
6
Select simulation id (default is 'basicsimulation'). Accepted characters are a-z, A-Z, 0-9, - and _

Select run description (optional)

Simulation messageservice.BasicSimulation started...
..
```

1.  输出将显示关于负载测试结果的某些统计数据。请求将被分类到小于 800 毫秒、800 毫秒到 1,200 毫秒之间以及超过 1,200 毫秒的几个类别。将显示一个 HTML 文件的链接。在浏览器中打开它，以查看关于你的负载测试的图表和其他有用的可视化。

正如我们在本食谱中看到的，Gatling 在运行负载测试方面提供了很多灵活性。通过使用 DSL 的一些巧妙脚本，可以更接近地模拟生产流量，通过解析日志文件和生成请求，根据延迟、响应或其他请求元素做出动态决策。Gatling 和 Vegeta 都是优秀的负载测试工具，你可以使用它们来探索你的系统在各种负载条件下的运行情况。

# 构建自动扩展集群

随着虚拟化和向基于云的基础设施的迁移，应用程序可以存在于弹性基础设施上，该基础设施根据预期的或测量的流量模式进行扩展和收缩。如果你的应用程序经历高峰期，你不需要在非高峰期配置全部容量，从而浪费计算资源和金钱。从虚拟化到容器和容器调度器，动态基础设施越来越普遍，这种基础设施会根据系统的需求进行变化。

微服务非常适合自动扩展。因为我们可以分别扩展系统的不同部分，所以更容易衡量特定服务及其依赖项的扩展需求。

创建自动扩展集群有许多方法。在下一章中，我们将讨论容器编排工具，但不会跳过，自动扩展集群也可以在任何云服务提供商中创建。在本菜谱中，我们将介绍使用 *Amazon Web Services* 创建自动扩展计算集群，特别是 Amazon EC2 自动扩展。我们将创建一个集群，其中包含多个运行在我们的消息服务背后的 EC2 实例，并位于 **应用程序负载均衡器** (**ALB**) 后面。我们将配置我们的集群，以便根据 CPU 利用率自动添加实例。

# 如何做到这一点...

让我们检查以下步骤：

1.  此菜谱需要 AWS 账户。如果您还没有 AWS 账户，请访问 [`aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/`](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) 创建一个账户，并在 [`docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html`](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) 创建一组访问密钥。安装 `aws cli` 工具。如果您在 OS X 上并且已安装 HomeBrew，可以使用以下命令完成此操作：

```js
$ brew install aws
```

1.  配置 `aws` 命令行工具，输入您创建的访问密钥：

```js
$ aws configure
```

1.  创建启动配置。启动配置是自动扩展组在创建新实例时使用的模板。在这种情况下，我们选择了 Amazon AMI 和 `t2.nano` 作为我们的 EC2 实例类型（有关更多详细信息，请参阅 [`aws.amazon.com/ec2/instance-types/`](https://aws.amazon.com/ec2/instance-types/)），如下面的代码所示：

```js
$ aws autoscaling create-launch-configuration --launch-configuration-name message-service-launch-configuration --image-id ari-f606f39f --instance-type t2.nano
```

1.  创建实际的自动扩展组。自动扩展组具有可配置的最大和最小大小，这指定了自动扩展组可以根据需求缩小或扩大的程度。在这种情况下，我们将创建一个具有最小 `1` 个实例和最大 `5` 个实例的自动扩展组，如下面的代码所示：

```js
$ aws autoscaling create-auto-scaling-group --auto-scaling-group-name message-service-asg --launch-configuration-name message-service-launch-configuration --max-size 5 --min-size 1 --availability-zones "us-east-1a"
```

1.  我们希望我们的自动扩展组中的实例可以通过负载均衡器访问，因此我们现在将创建它：

```js
$ aws elb create-load-balancer --load-balancer-name message-service-lb --listeners "Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=8082" --availability-zones us-east-1a

{
 "DNSName": "message-service-lb-1741394248.us-east-1.elb.amazonaws.com"
}
```

1.  为了自动扩展我们的自动扩展组，我们需要定义一个指标。集群可以根据内存、CPU 利用率或请求速率进行扩展。在这种情况下，我们将配置我们的扩展策略以使用 CPU 利用率。如果 CPU 利用率达到 20% 的平均值，我们的自动扩展组将创建更多实例。创建一个名为 `config.json` 的文件：

```js
{
  "TargetValue": 20.0,
  "PredefinedMetricSpecification":
    {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    }
}
```

1.  将扩展策略附加到我们的自动扩展组。

```js
$ aws autoscaling put-scaling-policy --policy-name cpu20 --auto-scaling-group-name message-service-asg --policy-type TargetTrackingScaling --target-tracking-configuration file://config.json
```

我们的自扩展组现在配置为当 CPU 利用率超过 20%的平均值时进行扩展。启动配置还可以包括安装和配置您的服务的引导步骤——通常使用某种配置管理工具，例如**Chef**或**Puppet**——或者它可以配置为从私有 Docker 仓库拉取 Docker 镜像。
