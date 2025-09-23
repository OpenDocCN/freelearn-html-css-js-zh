# 第七章：监控和可观察性

在本章中，我们将介绍以下配方：

+   结构化 JSON 记录

+   使用 StatsD 和 Graphite 收集指标

+   使用 Prometheus 收集指标

+   使用跟踪简化调试

+   当出现问题时发出警报

# 简介

微服务增加了架构的复杂性。随着系统中移动部件的增加，监控和观察系统的行为变得更加重要和更具挑战性。在微服务架构中，影响一个服务的故障条件可能会以意想不到的方式级联，影响整个系统。数据中心某个地方的故障交换机可能正在导致某个服务的异常高延迟，这可能导致来自 API 网关的请求间歇性超时，这可能导致意外的用户影响，从而触发警报。这种场景在微服务架构中并不罕见，需要提前考虑，以便工程师可以轻松确定影响客户的故障的性质。分布式系统注定会经历某些故障，必须特别考虑将可观察性构建到系统中。

微服务还迫使我们需要转向 DevOps。许多传统的监控解决方案是在操作是系统管理员或操作工程师特殊且独立群体唯一责任的时代开发的。系统管理员和操作工程师通常对系统级或主机级指标感兴趣，例如 CPU、内存、磁盘和网络使用情况。这些指标很重要，但只是可观察性的一小部分。**可观察性**也必须由编写微服务的工程师考虑。使用指标来观察系统特有的事件同样重要，例如抛出某些类型的异常或发送到队列的事件数量。

规划可观察性也为我们提供了在生产环境中有效测试系统所需的信息。用于预演和集成测试的临时环境可能很有用，但它们无法测试整个类别的故障状态。如第五章可靠性模式中所述，Gamedays 和其他形式的故障注入对于提高系统的弹性至关重要。可观察的系统适合这种测试，使工程师能够对我们的系统理解充满信心。

在本章中，我们将介绍监控和可观察性的几个原则。我们将演示如何修改我们的服务以生成结构化日志。我们还将查看指标，使用多个不同的系统来收集、聚合和可视化指标。最后，我们将探讨跟踪，这是一种查看请求如何穿越系统的各个组件的方法，并在检测到影响用户的错误条件时发出警报。

# 结构化 JSON 记录

输出有用的日志是构建可观察服务的关键部分。有用的日志构成是主观的，但一个好的指导原则是日志应包含关于系统关键事件的带时间戳的信息。一个好的日志系统支持可配置的日志级别概念，因此可以根据工程师的需求在特定时间内调整发送到日志的信息量。例如，当在生产环境中测试服务的故障场景时，提高日志级别并获取系统事件的更多详细信息可能是有用的。

Java 应用程序中最受欢迎的两个日志库是 **Log4j** ([`logging.apache.org/log4j/2.x/`](https://logging.apache.org/log4j/2.x/)) 和 **Logback** ([`logback.qos.ch/`](https://logback.qos.ch/))。默认情况下，这两个库将以非结构化格式输出日志条目，通常是空格分隔的字段，包括时间戳、日志级别和消息等信息。这很有用，但在微服务架构中尤其有用，其中多个服务可能向集中日志存储输出事件日志；输出具有一致性的结构化日志非常有用。

JSON 已经成为在系统间传递消息的通用标准。几乎每种流行的语言都有解析和生成 JSON 的库。它轻量级，但结构化，使其成为事件日志等数据的好选择。以 JSON 格式输出事件日志使得将服务日志输入集中存储并分析查询日志数据变得更加容易。

在这个菜谱中，我们将修改我们的消息服务以使用流行的 `logback` 库来生成 Java 应用程序的日志。

# 如何做到这一点...

让我们看看以下步骤：

1.  从 第六章，*安全*，打开消息服务项目。我们将做的第一个更改是向 `build.gradle` 文件中添加 `logback` 库：

```js
group 'com.packtpub.microservices'
version '1.0-SNAPSHOT'

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.springframework.boot', name: 'spring-boot-gradle-plugin', version: '1.5.9.RELEASE'
    }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web'
    compile group: 'io.github.resilience4j', name: 'resilience4j-circuitbreaker', version: '0.11.0'
 compile group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '4.7'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

1.  创建一个 `logback.xml` 配置文件。在配置文件中，我们将创建一个名为 `jsonLogger` 的单个记录器，它引用一个名为 `consoleAppender` 的单个追加器：

```js
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <logger name="jsonLogger" additivity="false" level="DEBUG">
        <appender-ref ref="consoleAppender"/>
    </logger>
    <root level="INFO">
        <appender-ref ref="consoleAppender"/>
    </root>
</configuration>
```

1.  将单个示例日志消息添加到 `Application.java` 以测试我们新的日志配置：

```js
package com.packtpub.microservices.ch07.message;

import com.packtpub.microservices.ch07.message.clients.SocialGraphClient;
import org.apache.log4j.LogManager;
import org.apache.log4j.Logger;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@SpringBootApplication
@EnableAsync
public class Application {

 private Logger logger = LogManager.getLogger(Application.class);

    @Bean
    public MessageRepository messageRepository() {
        return new MessageRepository();
    }

    @Bean
    public SocialGraphClient socialGraphClient() {
        return new SocialGraphClient("http://localhost:4567");
    }

    public static void main(String[] args) {
 logger.info("Starting application");
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(2);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("SocialServiceCall-");
        executor.initialize();
        return executor;
    }
}
```

1.  运行应用程序并查看日志消息现在以 JSON 格式输出：

```js
$ ./gradlew bootRun

> Task :bootRun
{"@timestamp":"2018-08-09T22:08:22.959-05:00","@version":1,"message":"Starting application","logger_name":"com.packtpub.microservices.ch07.message.Application","thread_name":"main","level":"INFO","level_value":20000}

 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
 '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

{"@timestamp":"2018-08-09T22:08:23.786-05:00","@version":1,"message":"Starting Application on fartlek.local with PID 82453 (/Users/posman/projects/microservices-cookbook/chapter07/message-service/build/classes/java/main started by posman in /Users/posman/projects/microservices-cookbook/chapter07/message-service)","logger_name":"com.packtpub.microservices.ch07.message.Application","thread_name":"main","level":"INFO","level_value":20000}
```

# 使用 StatsD 和 Graphite 收集指标

指标是随时间变化的数值测量。我们系统中收集的最常见的指标类型是计数器、计时器和仪表。计数器正是其名称的含义，在一定时间内增加一定次数的值。计时器可以用来测量系统中的重复事件，例如处理请求或执行数据库查询所需的时间。仪表只是可以记录的任意数值。

**StatsD** 是一个开源的网络守护进程，于 2011 年在 Etsy 发明。指标数据被推送到一个 `statsd` 服务器，通常在同一服务器上，在发送到持久后端之前聚合数据。与 `statsd` 一起使用最常用的后端之一是 **Graphite**，一个开源的时间序列存储引擎和绘图工具。Graphite 和 StatsD 一起构成了一个非常流行的指标栈。它们易于入门，拥有庞大的社区和大量的工具和库。

Spring Boot 有一个名为 **Actuator** 的子项目，它为服务添加了多个生产就绪功能。Actuator 为我们的服务免费提供了一些指标，与名为 micrometer 的项目一起，Actuator 实现了一个对各种指标后端的供应商中立 API。我们将在这个菜谱和下一个菜谱中使用 Actuator 和 micrometer。

在这个菜谱中，我们将向之前菜谱中使用的 message-service 添加 Actuator。我们将创建一些自定义指标，并演示如何使用 `statsd` 和 `graphite` 绘制应用程序的指标。我们将在本地 docker 容器中运行 `statsd` 和 `graphite`。

# 如何做到这一点……

让我们看看以下步骤：

1.  打开之前菜谱中的 message-service 项目。我们将升级 Spring Boot 的版本，并将 `actuator` 和 `micrometer` 添加到我们的依赖列表中。修改 `build.gradle` 文件，使其看起来如下所示：

```js
group 'com.packtpub.microservices'
version '1.0-SNAPSHOT'

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.springframework.boot', name: 'spring-boot-gradle-plugin', version: '2.0.4.RELEASE'
    }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web', version: '2.0.4.RELEASE'
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-actuator', version: '2.0.4.RELEASE'
    compile group: 'io.micrometer', name: 'micrometer-core', version: '1.0.6'
    compile group: 'io.micrometer', name: 'micrometer-registry-statsd', version: '1.0.6'
    compile group: 'io.github.resilience4j', name: 'resilience4j-circuitbreaker', version: '0.11.0'
    compile group: 'log4j', name: 'log4j', version: '1.2.17'
    compile group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '5.2'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

1.  在 `src/main/resources` 目录下打开 `application.yml` 文件，并添加以下内容：

```js
server:
  port:
    8082

management:
  metrics:
    export:
      statsd:
        enabled: true
        flavor: "etsy"
        host:
          0.0.0.0
        port:
          8125
```

1.  我们的应用程序现在支持向本地运行的 `statsd` 实例发出指标。打开 `MessageController.java` 文件，并将 `Timed` 注解添加到类以及 `get` 方法中：

```js
package com.packtpub.microservices.ch07.message.controllers;

import com.packtpub.microservices.ch07.message.MessageRepository;
import com.packtpub.microservices.ch07.message.clients.SocialGraphClient;
import com.packtpub.microservices.ch07.message.exceptions.MessageNotFoundException;
import com.packtpub.microservices.ch07.message.exceptions.MessageSendForbiddenException;
import com.packtpub.microservices.ch07.message.models.Message;
import com.packtpub.microservices.ch07.message.models.UserFriendships;
import io.micrometer.core.annotation.Timed;
import io.micrometer.statsd.StatsdMeterRegistry;
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
@Timed
public class MessageController {

    @Autowired
    private MessageRepository messagesStore;

    @Autowired
    private SocialGraphClient socialGraphClient;

    @Autowired
    private StatsdMeterRegistry registry;

    @Timed(value="get.messages")
    @RequestMapping(path = "/{id}", method = RequestMethod.GET, produces = "application/json")
    public Message get(@PathVariable("id") String id) throws MessageNotFoundException {
        registry.counter("get_messages").increment();
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

1.  为了证明指标实际上正在发出，我们将在本地 docker 容器中运行 `statsd` 和 graphite。安装了 `docker` 后，运行以下命令，它将从 `dockerhub` 拉取一个镜像并在本地运行一个容器：

```js
 docker run -d --name graphite --restart=always \
 -p 80:80 -p 2003-2004:2003-2004 -p 2023-2024:2023-2024 \
 -p 8125:8125/udp -p 8126:8126 \
 hopsoft/graphite-statsd
```

1.  现在，访问 `http://localhost` 来查看你的指标！

# 使用 Prometheus 收集指标

**Prometheus** 是一个开源的监控和警报工具包，最初于 2012 年在 **SoundCloud** 开发。它受到了谷歌的 Borgmon 的启发。与 `statsd` 等系统采用的推送模型不同，Prometheus 使用拉模型来收集指标。不是每个服务都负责将指标推送到 `statsd` 服务器，而是 Prometheus 负责抓取具有指标的服务公开的端点。这种责任反转在按比例操作指标时提供了一些好处。Prometheus 中的目标可以手动配置或通过服务发现配置。

与使用分层格式存储指标数据的系统（如 Graphite）相比，Prometheus 采用多维数据模型。Prometheus 中的时间序列数据由一个指标名称（例如 `http_request_duration_seconds`）和一个或多个标签（例如 `service=message-service` 和 `method=POST`）标识。这种格式可以更容易地在多个不同的应用程序之间标准化指标，这在微服务架构中尤其有价值。

在这个菜谱中，我们将继续使用 message-service 以及 Actuator 和 micrometer 库。我们将配置 micrometer 以使用 Prometheus 指标注册表，并公开 Prometheus 可以抓取以收集我们服务指标的端点。然后我们将配置 Prometheus 以抓取运行在本地的 message-service，并在本地运行 Prometheus 以验证我们是否可以查询我们的指标。

# 如何操作...

1.  打开消息服务并编辑 `build.gradle` 文件，以包含 actuator 和 micrometer-prometheus 依赖项：

```js
group 'com.packtpub.microservices'
version '1.0-SNAPSHOT'

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.springframework.boot', name: 'spring-boot-gradle-plugin', version: '2.0.4.RELEASE'
    }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web', version: '2.0.4.RELEASE'
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-actuator', version: '2.0.4.RELEASE'
    compile group: 'io.micrometer', name: 'micrometer-core', version: '1.0.6'
    compile group: 'io.micrometer', name: 'micrometer-registry-prometheus', version: '1.0.6'
    compile group: 'io.github.resilience4j', name: 'resilience4j-circuitbreaker', version: '0.11.0'
    compile group: 'log4j', name: 'log4j', version: '1.2.17'
    compile group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '5.2'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

1.  将以下内容添加到 `application.yml` 中。这将启用一个端点，该端点公开 Prometheus 指标注册表中收集的指标。请注意，我们正在为 `actuator` 添加的管理端点打开另一个端口：

```js
server:
  port:
    8082

management:
  server:
    port:
      8081
  endpoint:
    metrics:
      enabled: true
    prometheus:
      enabled: true
  endpoints:
    web:
      base-path: "/manage"
      exposure:
        include: "*"
  metrics:
    export:
      prometheus:
        enabled: true
```

1.  我们现在可以测试我们的服务是否在 `/manage/prometheus` 端点公开指标。运行服务并执行以下 `curl` 请求：

```js
$ curl http://localhost:8081/manage/prometheus

# HELP tomcat_global_request_seconds
# TYPE tomcat_global_request_seconds summary
tomcat_global_request_seconds_count{name="http-nio-8082",} 0.0
tomcat_global_request_seconds_sum{name="http-nio-8082",} 0.0
# HELP tomcat_sessions_active_max
# TYPE tomcat_sessions_active_max gauge
tomcat_sessions_active_max 0.0
# HELP process_uptime_seconds The uptime of the Java virtual machine
# TYPE process_uptime_seconds gauge
process_uptime_seconds 957.132
# HELP jvm_gc_live_data_size_bytes Size of old generation memory pool after a full GC
# TYPE jvm_gc_live_data_size_bytes gauge
jvm_gc_live_data_size_bytes 1.9244032E7
```

1.  在 `/tmp` 目录中创建一个名为 `prometheus.yml` 的新文件，其中包含有关我们的目标信息，并配置和运行 Prometheus 在 Docker 容器中：

```js
# my global config
global:
 scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
 evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
 # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
 alertmanagers:
 - static_configs:
 - targets:
 # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
 # - "first_rules.yml"
 # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
 # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
 - job_name: 'prometheus'

 # metrics_path defaults to '/metrics'
 # scheme defaults to 'http'.

 static_configs:
 - targets: ['localhost:9090']

 - job_name: 'message-service'
 metrics_path: '/manage/prometheus'
 static_configs:
 - targets: ['localhost:8081']
```

1.  下载并解压适用于您平台的 Prometheus 版本。说明在 Prometheus 网站上（[`prometheus.io/docs/introduction/first_steps/`](https://prometheus.io/docs/introduction/first_steps/)）。使用我们在上一步创建的配置文件运行 Prometheus：

```js
$ ./prometheus --config.file=/tmp/prometheus.yml
```

1.  在您的浏览器中打开 `http://localhost:9090` 以执行 Prometheus 查询并查看您的指标！在您开始向服务发送请求之前，您将看到的唯一指标将是 JVM 和系统指标，但这应该能给您一个关于您可以使用 Prometheus 进行何种查询以及演示抓取器如何工作的概念。

# 使用跟踪简化调试

在微服务架构中，单个请求可以经过几个不同的服务，并导致写入几个不同的数据存储和事件队列。在生产事件调试时，并不总是清楚问题是否存在于一个系统或另一个系统中。这种缺乏具体性意味着指标和日志只是整个画面的一小部分。有时我们需要放大并查看从用户代理到终端服务再到用户代理的整个请求生命周期。

在 2010 年，谷歌的工程师发表了一篇论文，描述了**Dapper** ([`research.google.com/archive/papers/dapper-2010-1.pdf`](https://research.google.com/archive/papers/dapper-2010-1.pdf))，这是一个大规模分布式系统跟踪基础设施。论文中描述了谷歌如何使用内部开发的跟踪系统来帮助观察系统行为和调试性能问题。这项工作启发了其他人，包括 Twitter 的工程师，他们在 2012 年引入了一个名为**Zipkin** ([`blog.twitter.com/engineering/en_us/a/2012/distributed-systems-tracing-with-zipkin.html`](https://blog.twitter.com/engineering/en_us/a/2012/distributed-systems-tracing-with-zipkin.html))的开源分布式跟踪系统。Zipkin 最初是 Dapper 论文的一个实现，但后来发展成了一套完整的性能分析和检查 Twitter 基础设施请求的工具。

在跟踪领域进行的所有工作都明显表明需要某种标准化的 API。**OpenTracing** ([`opentracing.io/`](http://opentracing.io/))框架正是为此而做的尝试。OpenTracing 定义了一个规范，详细说明了跟踪的跨语言标准。许多不同公司的工程师都为此做出了贡献，包括最初创建 Jaeger ([`eng.uber.com/distributed-tracing/`](https://eng.uber.com/distributed-tracing/))的 Uber 工程师，Jaeger 是一个符合 OpenTracing 规范的开放源代码、端到端分布式跟踪系统。

在这个菜谱中，我们将修改我们的 message-service 以添加跟踪支持。然后，我们将在 docker 容器中运行 Jaeger，这样我们就可以在实践中看到一些跟踪信息。

# 如何做到这一点...

1.  打开 message-service 项目，并用以下内容替换`build.gradle`的内容：

```js
group 'com.packtpub.microservices'
version '1.0-SNAPSHOT'

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.springframework.boot', name: 'spring-boot-gradle-plugin', version: '2.0.4.RELEASE'
    }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web', version: '2.0.4.RELEASE'
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-actuator', version: '2.0.4.RELEASE'
    compile group: 'io.micrometer', name: 'micrometer-core', version: '1.0.6'
    compile group: 'io.micrometer', name: 'micrometer-registry-statsd', version: '1.0.6'
    compile group: 'io.opentracing.contrib', name: 'opentracing-spring-cloud-starter-jaeger', version: '0.1.13'
    compile group: 'io.github.resilience4j', name: 'resilience4j-circuitbreaker', version: '0.11.0'
    compile group: 'log4j', name: 'log4j', version: '1.2.17'
    compile group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '5.2'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

1.  在`src/main/resources`目录中打开`application.yml`文件，并添加一个`opentracing`配置部分。在这里，我们正在配置我们的`opentracing`实现以连接到本地运行的端口`6831`上的 Jaeger 实例：

```js
opentracing:
  jaeger:
    udp-sender:
      host: "localhost"
      port:
        6831

spring:
  application:
    name: "message-service"
```

1.  为了收集跟踪信息，我们将在本地运行一个 Jaeger 实例。Docker 使用以下命令使这变得简单：

```js
docker run -d --name jaeger \
 -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
 -p 5775:5775/udp \
 -p 6831:6831/udp \
 -p 6832:6832/udp \
 -p 5778:5778 \
 -p 16686:16686 \
 -p 14268:14268 \
 -p 9411:9411 \
 jaegertracing/all-in-one:latest
```

1.  运行 message-service 并发出一些示例请求（即使它们导致 404）。在浏览器中打开`http://localhost:16686`，你会看到 Jaeger 的 Web 界面。点击搜索并探索迄今为止收集到的跟踪数据！

# 当出现问题时提醒我们

如果你正在认真考虑微服务，你可能正在运行一个 24/7 的服务。客户要求你的服务在任何时候都可以使用。将这种对可用性的需求增加与分布式系统不断经历某种故障的现实进行对比。没有系统是完全健康的。

无论你是采用单体架构还是微服务架构，试图完全避免生产事故都是没有意义的。相反，你应该尝试优化你应对故障的方式，通过减少解决故障所需的时间来限制其对客户的影响。

减少解决事故所需的时间（通常以平均解决时间或 MTTR 衡量）首先需要减少**平均检测时间**（**MTTD**）。在服务处于影响客户的故障状态时，能够准确地向值班工程师发出警报对于保持正常运行至关重要。好的警报应该是可操作的且紧急的；如果你的系统在故障不可操作或非紧急（不影响客户）时通知值班工程师，你可能会让值班工程师疲惫不堪，并造成通常所说的告警疲劳。告警疲劳是真实存在的，并且可能对正常运行时间产生比任何数量的软件错误或故障硬件更灾难性的影响。持续改进你的系统告警，以获得阈值和其他因素的恰到好处，防止误报，同时保持对真正影响客户的故障的告警。

告警基础设施不是你想自己构建的东西。**PagerDuty**是一个 SaaS 工具，允许你为特定服务的值班工程师团队创建升级策略和日程安排。使用 PagerDuty，你可以设置一个轮换日程，例如，一个五人团队的工程师可以预期每五周值班一周。升级策略允许你在值班工程师不可用的情况下（例如，他们可能在高速公路上开车）配置一系列步骤。升级策略通常配置为在事故未被确认一定时间后，向二级值班日程、经理甚至整个团队发送警报。使用像 PagerDuty 这样的系统，可以让团队中的工程师在享受必要的非值班时间的同时，知道影响客户的故障将会得到及时响应。

可以使用任何数量的支持集成手动配置警报，但这既耗时又容易出错。相反，我们希望有一个系统，允许您自动化创建和维护服务的警报。本章中涵盖的 Prometheus 监控和警报工具包包括一个名为 Alertmanager 的工具，它允许您做到这一点。在这个配方中，我们将修改我们的消息服务以使用 Alertmanager 添加警报。具体来说，我们将配置一个警报，当平均响应时间超过 500 毫秒且至少持续 5 分钟时触发。我们将从已经包含 Prometheus 指标的消息服务版本开始工作。在这个配方中，我们不会添加任何 PagerDuty 集成，因为这需要 PagerDuty 账户才能继续。PagerDuty 在其网站上有一个优秀的集成指南。我们将配置 `alertmanager` 以发送基于简单 WebHook 的警报。

# 如何操作...

现在，让我们看看以下步骤：

1.  在之前的配方中，我们使用名为 `prometheus.yml` 的文件配置了 Prometheus。我们需要将 `alertmanager` 配置添加到该文件中，所以请再次打开它并添加以下内容：

```js
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
 alertmanagers:
 - static_configs:
 - targets:
 - localhost:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
 - "rules.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']

  - job_name: 'message-service'
    metrics_path: '/manage/prometheus'
    static_configs:
    - targets: ['localhost:8081']
```

1.  创建一个名为 `/tmp/rules.yml` 的新文件。该文件定义了我们希望 Prometheus 能够创建警报的规则：

```js
groups:
- name: message-service-latency
  rules:
  - alert: HighLatency
    expr: rate(http_server_requests_seconds_sum{job="message-service", instance="localhost:8081"}[1m]) / rate(http_server_requests_seconds_count{job="message-service", instance="localhost:8081"}[1m]) > .5
    for: 1m
    labels:
      severity: 'critical'
    annotations:
      summary: High request latency
```

1.  创建一个名为 `/tmp/alertmanager.yml` 的新文件。这是描述我们的警报配置的文件。它被分成几个不同的部分，这些部分是某些配置选项的全局集合，这些选项会影响 `alertmanager` 的工作方式。名为 receivers 的部分是我们配置警报通知系统的位置。在这种情况下，它是一个运行在本地的服务的 WebHook。这只是为了演示目的；我们将编写一个小的 Ruby 脚本，该脚本监听 HTTP 请求并将有效负载打印到标准输出：

```js
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:4567/'
```

1.  这是将打印出我们的警报的小型 Ruby 服务的源代码：

```js
require 'sinatra'

post '/' do
    body = request.body.read()
    puts body
    return body
end
```

1.  运行 Ruby 脚本，重启 `prometheus` 和 `alertmanager`。这三个系统运行后，我们将准备好测试我们的警报：

```js
$ ruby echo.rb
...

$ ./prometheus --config.file=/tmp/prometheus.yml

$ ./alertmanager --config.file=/tmp/alertmanager.yml
...
```

1.  为了使我们的警报触发，打开消息服务并添加以下行到 `MessageController.java`。这是一行代码，它将迫使控制器在返回响应之前暂停 600 毫秒。请注意，这超出了我们在规则配置中描述的阈值：

```js
@RequestMapping(path = "/{id}", method = RequestMethod.GET, produces = "application/json")
public Message get(@PathVariable("id") String id) throws MessageNotFoundException {

 try { Thread.sleep(600); } catch (InterruptedException e) } e.printStackTrace(); } 
    return messagesStore.get(id);
}
```

1.  在设置完成后，运行您更新的消息服务并向其发送多个请求。一分钟过后，Prometheus 应该会通知 Alertmanager，然后 Alertmanager 应该会通知您本地的调试 Ruby 服务。您的警报正在工作！
