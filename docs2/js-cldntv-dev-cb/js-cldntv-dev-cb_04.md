# 利用云端的边缘优势

本章将涵盖以下食谱：

+   从 CDN 提供单页应用程序

+   将自定义域名与 CDN 关联

+   从 CDN 提供网站

+   在 CDN 后部署服务

+   从 CDN 提供静态 JSON

+   触发 CDN 中内容的无效化

+   在云端的边缘执行代码

# 简介

云端的边缘可能是云原生系统中最被低估的一层。然而，正如我之前提到的，我的第一个云原生“哇”时刻是当我意识到我可以在不运行弹性负载均衡器和多个 EC2 实例的复杂度、风险和成本的情况下，从边缘运行 **单页应用程序**（SPA）表示层。此外，利用边缘可以将云原生系统的一些方面扩展到全球规模，而无需在多区域部署上付出额外的努力。我们的最终用户享受较低的延迟，而我们减少了系统内部负载，降低了成本，并提高了安全性。本章中的食谱涵盖了多种我们可以利用云端的边缘来提升云原生系统质量的方法，而所需的工作量最小。

# 从 CDN 提供单页应用程序

在 *部署单页应用程序* 食谱中，我们介绍了从 S3 存储桶提供单页应用程序所需的步骤。当一位架构师意识到这样一个简单的部署过程可以提供如此多的可扩展性时，这真是一件令人愉快的事情。以下是来自我的客户的一个最近的引用——“这就完了？真的吗？这就完了？”在本食谱中，我们将此过程进一步推进，并展示我们如何轻松地在 S3 前添加 CloudFront **内容分发网络**（CDN）层，以利用更多有价值的特性。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch4/cdn-spa --path cncb-cdn-spa
```

1.  使用 `cd cncb-cdn-spa` 切换到 `cncb-cdn-spa` 目录。

1.  查看以下内容的 `serverless.yml` 文件：

```js
service: cncb-cdn-spa

provider:
  name: aws

plugins:
  - serverless-spa-deploy
  - serverless-spa-config

custom:
  spa:
    files:
      - source: ./build
        globs: '**/*'
        headers:
          CacheControl: max-age=31536000 # 1 year
      - source: ./build
        globs: 'index.html'
        headers:
          CacheControl: max-age=300 # 5 minutes
```

1.  使用 `npm install` 安装依赖。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容：

```js
{
 "Resources": {
    ...
    "WebsiteBucket": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "AccessControl": "PublicRead",
        "WebsiteConfiguration": {
          "IndexDocument": "index.html",
          "ErrorDocument": "index.html"
        }
      }
    },
    "WebsiteDistribution": {
      "Type": "AWS::CloudFront::Distribution",
      "Properties": {
        "DistributionConfig": {
          "Comment": "Website: test-cncb-cdn-spa (us-east-1)",
          "Origins": [
            {
              "Id": "S3Origin",
              "DomainName": {
                "Fn::Select": [
                  1,
                  {
                    "Fn::Split": [
                      "//",
                      {
                        "Fn::GetAtt": [
                          "WebsiteBucket",
                          "WebsiteURL"
                        ]
                      }
                    ]
                  }
                ]
              },
              "CustomOriginConfig": {
                "OriginProtocolPolicy": "http-only"
              }
            }
          ],
          "Enabled": true,
          "PriceClass": "PriceClass_100",
          "DefaultRootObject": "index.html",
          "CustomErrorResponses": [
            {
              "ErrorCachingMinTTL": 0,
              "ErrorCode": 404,
              "ResponseCode": 200,
              "ResponsePagePath": "/index.html"
            }
          ],
          "DefaultCacheBehavior": {
            "TargetOriginId": "S3Origin",
            "AllowedMethods": [
              "GET",
              "HEAD"
            ],
            "CachedMethods": [
              "HEAD",
              "GET"
            ],
            "Compress": true,
            "ForwardedValues": {
              "QueryString": false,
              "Cookies": {
                "Forward": "none"
              }
            },
            "MinTTL": 0,
            "DefaultTTL": 0,
            "ViewerProtocolPolicy": "allow-all"
          }
        }
      }
    }
  },
  ...
}
```

1.  使用 `npm run build` 构建应用。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cdn-spa@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cdn-spa
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
Stack Outputs
WebsiteDistributionURL: https://dqvo8ga8z7ao3.cloudfront.net
WebsiteS3URL: http://cncb-cdn-spa-john-websitebucket-1huxcgjseaili.s3-website-us-east-1.amazonaws.com
WebsiteBucketName: cncb-cdn-spa-john-websitebucket-1huxcgjseaili
WebsiteDistributionId: E3JF634XQF4PE9
...
Serverless: Path: ./build
Serverless: File: asset-manifest.json (application/json)
...
```

部署 CloudFront 分发可能需要 20 分钟或更长时间。

1.  在 AWS 控制台中查看堆栈和 CloudFront 分发。

1.  浏览到堆栈输出中显示的 `WebsiteS3URL` 和 `WebsiteDistributionURL`，并在浏览器检查工具的网络标签中注意启用和禁用缓存时两者的性能差异：

```js
http://<see WebsiteS3URL output>.s3-website-us-east-1.amazonaws.com
http://<see WebsiteDistributionURL output>.cloudfront.net
```

1.  使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈，一旦你完成操作。

# 它是如何工作的...

CloudFront 分发的配置既冗长又模板化。`serverless-spa-config`插件简化了这项工作，封装了最佳实践，并允许通过异常进行配置。在这个菜谱中，我们使用所有默认设置。在生成的`.serverless/cloudformation-template-update-stack.json`模板中，我们可以看到`WebsiteBucket`被定义并配置为默认的`S3Origin`，其中`index.html`文件作为`DefaultRootObject`。`PriceClass`默认为北美和欧洲，以最小化分配分发所需的时间。桶的错误处理（`ErrorDocument`）和分发的错误处理（`CustomErrorResponses`）被配置为委托错误处理给 SPA 的逻辑。

分发的核心目的是在边缘缓存 SPA 资源。这个逻辑由两部分处理。首先，`DefaultCacheBehavior`被设置为`DefaultTTL`为零，以确保使用桶中个别资源的缓存控制头来控制 TTL。其次，`serverless-spa-deploy`插件配置了两个不同的`CacheControl`设置。除了`index.html`文件之外的所有内容都部署了一个最大存活时间（max-age）为一年，因为 Webpack 使用为每个构建生成的哈希值来命名这些资源，从而隐式地清除缓存。`index.html`文件必须有一个恒定的名称，因为它作为`DefaultRootObject`，所以我们将其 max-age 设置为 5 分钟。这意味着在部署后的大约五分钟内，我们可以预期最终用户开始接收最新的代码更改。五分钟后，浏览器将向 CDN 请求`index.html`文件，如果 ETag 未更改，CDN 将返回错误 304。这在不减少数据传输的同时允许快速传播更改。您可以根据需要增加或减少 max-age。

在这个阶段，我们通过将资源推送到边缘以减少延迟，并压缩资源以减少带宽，使用 CDN 来提高用户的下载性能。仅此一项就足以利用云的边缘。其他功能包括自定义域名、SSL 证书、日志记录以及与**Web 应用防火墙（WAF**）的集成。我们将在后续的菜谱中介绍这些主题。

# 将自定义域名与 CDN 关联

在“从 CDN 中提供单页应用程序”菜谱中，我们介绍了在 S3 前面添加 CloudFront CDN 层并利用更多有价值功能的步骤。这些功能之一是能够将自定义域名与 CDN 提供的资源关联起来，例如单页应用程序或云原生服务。在这个菜谱中，我们将部署过程再向前推进一步，并演示如何添加 Route53 记录集和 CloudFront 别名。

# 准备工作

您需要一个已注册的域名和一个 Route53 托管区域，您可以使用这些信息在本配方中创建一个用于部署的 SPA 子域名。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch4/cdn-dns --path cncb-cdn-dns
```

1.  使用 `cd cncb-cdn-dns` 命令进入 `cncb-cdn-dns` 目录。

1.  查看以下内容的 `serverless.yml` 文件：

```js
service: cncb-cdn-dns

provider:
  name: aws

plugins:
  - serverless-spa-deploy
  - serverless-spa-config

custom:
  spa:
    files:
      ...   
  dns:
    hostedZoneId:  Z1234567890123
    domainName: example.com
    endpoint: app.${self:custom.dns.domainName}
```

1.  使用您的 `hostedZoneId` 和 `domainName` 更新 `serverless.yml` 文件。

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容：

```js
{
  "Resources": {
    ...
    "WebsiteDistribution": {
      "Type": "AWS::CloudFront::Distribution",
      "Properties": {
        "DistributionConfig": {
          ...
          "Aliases": [
            "app.example.com"
          ],
          ...
          }
        }
      }
    },
    "WebsiteEndpointRecord": {
      "Type": "AWS::Route53::RecordSet",
      "Properties": {
        "HostedZoneId": "Z1234567890123",
        "Name": "app.example.com.",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": {
            "Fn::GetAtt": [
              "WebsiteDistribution",
              "DomainName"
            ]
          }
        }
      }
    }
  },
  ...
}
```

1.  使用 `npm run build` 构建应用程序。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cdn-dns@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cdn-dns
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
Stack Outputs
WebsiteDistributionURL: https://dwrdvnqsnetay.cloudfront.net
WebsiteS3URL: http://cncb-cdn-dns-john-websitebucket-1dwzb5bfkv34s.s3-website-us-east-1.amazonaws.com
WebsiteBucketName: cncb-cdn-dns-john-websitebucket-1dwzb5bfkv34s
WebsiteURL: http://app.example.com
WebsiteDistributionId: ED6YKAFJDF2ND
...
```

部署 CloudFront 分发可能需要 20 分钟或更长时间。

1.  在 AWS 控制台中查看堆栈、Route53 和 CloudFront 分发。

1.  浏览到堆栈输出中显示的 `WebsiteURL`：

```js
http://app.example.com <see WebsiteURL output>
```

1.  完成后使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

在本配方中，我们通过为 SPA 添加自定义域名来改进 *从 CDN 提供单页应用程序* 配方。在 `serverless.yml` 中，我们添加了 `hostedZoneId`、`domainName` 和 `endpoint` 值。这些值触发 `serverless-spa-config` 插件配置 Route53 中的 `WebsiteEndpointRecord`，并在 CloudFront 分发上设置 `Aliases`。

# 从 CDN 提供网站服务

作为行业，我们倾向于创建非常动态的网站，即使内容不经常变化。一些 **内容管理系统** (**CMS**) 即使内容没有变化，也会为每个请求重新计算内容。这些请求通过多个层级，从数据库读取，然后计算并返回响应。平均响应时间在五秒左右并不罕见。据说，重复做同样的事情并期望不同的结果，这就是疯狂的定义。

云原生系统在创建网站方面采取了一种完全不同的方法。JAMstack ([`jamstack.org`](https://jamstack.org)) 是一种基于客户端 JavaScript、可重用 API 和标记的现代云原生方法。这些静态网站由 Git 工作流程管理，由 CI/CD 管道生成，并且每天部署到云的边缘多次。这是云原生挑战我们重新配置软件工程大脑的另一个例子。本配方演示了这些静态网站的生成和部署。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch4/cdn-site --path cncb-cdn-site
```

1.  使用 `cd cncb-cdn-site` 命令进入 `cncb-cdn-site` 目录。

1.  查看以下内容的 `serverless.yml` 文件：

```js
service: cncb-cdn-site

provider:
  name: aws

plugins:
  - serverless-spa-deploy
  - serverless-spa-config

custom:
  spa:
    files:
      - source: ./dist
        globs: '**/*'
        headers:
          CacheControl: max-age=31536000 # 1 year
    redirect: true
  dns:
    hostedZoneId:  Z1234567890123
    domainName: example.com
    endpoint: www.${self:custom.dns.domainName}
```

1.  使用您的 `hostedZoneId` 和 `domainName` 更新 `serverless.yml` 文件。

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容：

```js
{
  "Resources": {
    ...
    "RedirectBucket": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName": "example.com",
        "WebsiteConfiguration": {
          "RedirectAllRequestsTo": {
            "HostName": "www.example.com"
          }
        }
      }
    },
    "WebsiteDistribution": {
      "Type": "AWS::CloudFront::Distribution",
      "Properties": {
        "DistributionConfig": {
 "Comment": "Website: test-cncb-cdn-site (us-east-1)",
        ...
          "Aliases": [
            "www.example.com"
          ],
        ...
        }
      }
    },
    "RedirectDistribution": {
      "Type": "AWS::CloudFront::Distribution",
      "Properties": {
        "DistributionConfig": {
          "Comment": "Redirect: test-cncb-cdn-site (us-east-1)",
        ...
          "Aliases": [
            "example.com"
          ],
        ...
        }
      }
    },
    "WebsiteEndpointRecord": {
      "Type": "AWS::Route53::RecordSet",
      "Properties": {
        "HostedZoneId": "Z1234567890123",
        "Name": "www.example.com.",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": {
            "Fn::GetAtt": [
              "WebsiteDistribution",
              "DomainName"
            ]
          }
        }
      }
    },
    "RedirectEndpointRecord": {
      "Type": "AWS::Route53::RecordSet",
      "Properties": {
        "HostedZoneId": "Z1234567890123",
        "Name": "example.com.",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": {
            "Fn::GetAtt": [
              "RedirectDistribution",
              "DomainName"
            ]
          }
        }
      }
    }
  },
  ...
}
```

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cdn-site@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cdn-site
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
Stack Outputs
WebsiteDistributionURL: https://d2ooxtd49ayyfd.cloudfront.net
WebsiteS3URL: http://cncb-cdn-site-john-websitebucket-mrn9ntyltxim.s3-website-us-east-1.amazonaws.com
WebsiteBucketName: cncb-cdn-site-john-websitebucket-mrn9ntyltxim
WebsiteURL: http://www.example.com
WebsiteDistributionId: E10ZLN9USZTDSO
...
Serverless: Path: ./dist
Serverless: File: after/c2Vjb25kLXBvc3Q=/index.html (text/html)
Serverless: File: after/dGhpcmQtcG9zdA==/index.html (text/html)
Serverless: File: after/Zm91cnRoLXBvc3Q=/index.html (text/html)
Serverless: File: after/ZmlmdGgtcG9zdA==/index.html (text/html)
Serverless: File: after/Zmlyc3QtcG9zdA==/index.html (text/html)
Serverless: File: blog/fifth-post/index.html (text/html)
Serverless: File: blog/first-post/index.html (text/html)
Serverless: File: blog/fourth-post/index.html (text/html)
Serverless: File: blog/second-post/index.html (text/html)
Serverless: File: blog/third-post/index.html (text/html)
Serverless: File: favicon.ico (image/x-icon)
Serverless: File: index.html (text/html)
Serverless: File: phenomic/content/posts/by-default/1/desc/date/limit-2.json (application/json)
Serverless: File: phenomic/content/posts/by-default/1/desc/date/limit-2/after-c2Vjb25kLXBvc3Q=.json (application/json)
Serverless: File: phenomic/content/posts/by-default/1/desc/date/limit-2/after-dGhpcmQtcG9zdA==.json (application/json)
Serverless: File: phenomic/content/posts/by-default/1/desc/date/limit-2/after-Zm91cnRoLXBvc3Q=.json (application/json)
Serverless: File: phenomic/content/posts/by-default/1/desc/date/limit-2/after-ZmlmdGgtcG9zdA==.json (application/json)
Serverless: File: phenomic/content/posts/by-default/1/desc/date/limit-2/after-Zmlyc3QtcG9zdA==.json (application/json)
Serverless: File: phenomic/content/posts/item/fifth-post.json (application/json)
Serverless: File: phenomic/content/posts/item/first-post.json (application/json)
Serverless: File: phenomic/content/posts/item/fourth-post.json (application/json)
Serverless: File: phenomic/content/posts/item/second-post.json (application/json)
Serverless: File: phenomic/content/posts/item/third-post.json (application/json)
Serverless: File: phenomic/phenomic.main.9a7f8f5f.js (application/javascript)
Serverless: File: robots.txt (text/plain)
```

部署 CloudFront 分发可能需要 20 分钟或更长时间。

1.  在 AWS 控制台中查看堆栈、Route53 和 CloudFront 分发。

1.  浏览到堆栈输出中显示的`WebsiteURL`，然后再不带`www.`的情况下浏览，并观察重定向到`www`：

```js
http://www.example.com <see WebsiteURL output>
http://example.com <see WebsiteURL output>  
```

1.  浏览网站页面。

1.  完成使用`npm run rm:lcl -- -s $MY_STAGE`后，删除堆栈。

# 它是如何工作的...

在这个菜谱中，我们使用了一个名为**Phenomic**的静态站点生成器（[`www.staticgen.com/phenomic`](https://www.staticgen.com/phenomic)）。在**StaticGen**网站上列出了看似无穷无尽的工具（[`www.staticgen.com`](https://www.staticgen.com)）。使用 Phenomic，我们用 ReactJS 和 Markdown 的组合编写网站。然后，我们将 JavaScript 和 Markdown 同构生成一组静态资源，并将其部署到 S3，如部署输出所示。我已经预先构建了网站，并将生成的资源包含在存储库中。一个典型的 SPA 会在浏览器中下载 JavaScript 并生成网站。根据网站的不同，这个过程可能很耗时且明显。基于运行时的同构生成在将网站返回浏览器之前在服务器端执行此过程。另一方面，JAMstack 网站在部署时静态生成，这导致了最佳的运行时性能。尽管网站是静态的，但页面包含像任何 ReactJS SPA 一样编写的动态 JavaScript。这些网站经常更新并每天多次重新部署，这在不要求每次请求都询问服务器是否有所变化的情况下，又增加了一个动态维度。

这些网站通常部署到`www`子域名，并支持将根域名重定向到该子域名。基于**从 CDN 提供单页应用程序**和**将自定义域名与 CDN 关联**的菜谱，这个菜谱利用了`serverless-spa-deploy`和`serverless-spa-config`插件，并添加了一些额外的设置。我们指定`endpoint`为`www.${self:custom.dns.domainName}`，并将重定向标志设置为`true`。这反过来创建了`RedirectBucket`，它执行重定向，以及额外的`RedirectDistribution`和`RedirectEndpointRecord`以支持根域名。

# 在 CDN 后面部署服务

当我们想到 CDN 时，通常会认为它们仅用于提供静态内容。然而，将 CloudFront 放置在所有资源之前，即使是动态服务，也是 AWS 的最佳实践，以提高安全性和性能。从安全角度来看，CloudFront 最小化了攻击面，并在边缘处理 DDoS 攻击，以减轻内部组件的负载。至于性能，CloudFront 优化了边缘和可用区之间的管道，即使对于 `POST` 和 `PUT` 操作也能提高性能。此外，即使只有几秒钟的缓存控制头也能对 `GET` 操作产生重大影响。在本菜谱中，我们将演示如何将 CloudFront 添加到 AWS API Gateway 之前。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch4/cdn-service --path cncb-cdn-service
```

1.  使用 `cd cncb-cdn-service` 命令进入 `cncb-cdn-service` 目录。

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-cdn-service

provider:
  name: aws
  runtime: nodejs8.10
  endpointType: REGIONAL

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: hello
          method: get
          cors: true

resources:
  Resources:
    ApiDistribution:
      Type: AWS::CloudFront::Distribution
      Properties:
        DistributionConfig:
          ...
          Origins:
            - Id: ApiGateway
              DomainName:
                Fn::Join:
                  - "."
                  - - Ref: ApiGatewayRestApi
                    - execute-api.${opt:region}.amazonaws.com
              OriginPath: /${opt:stage}
              CustomOriginConfig:
                OriginProtocolPolicy: https-only
                OriginSSLProtocols: [ TLSv1.2 ]
          ...
          DefaultCacheBehavior:
            TargetOriginId: ApiGateway
            AllowedMethods: [ DELETE, GET, HEAD, OPTIONS, PATCH, POST, PUT ]
            CachedMethods: [ GET, HEAD, OPTIONS ]
            Compress: true
            ForwardedValues:
              QueryString: true
              Headers: [ Accept, Authorization ]
              Cookies:
                Forward: all
            MinTTL: 0
            DefaultTTL: 0
            ViewerProtocolPolicy: https-only

  Outputs:
    ApiDistributionId:
      Value:
        Ref: ApiDistribution  
    ApiDistributionEndpoint:
      Value:
        Fn::Join:
          - ''
          - - https://
            - Fn::GetAtt: [ ApiDistribution, DomainName ]
```

1.  查看名为 `handler.js` 的文件，其内容如下：

```js
module.exports.hello = (request, context, callback) => {
  const response = {
    statusCode: 200,
    headers: {
      'Cache-Control': 'max-age=5',
    },
    body: ...,
  };

  callback(null, response);
};
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cdn-service@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cdn-service
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
endpoints:
  GET - https://5clnzj3knc.execute-api.us-east-1.amazonaws.com/john/hello
functions:
  hello: cncb-cdn-service-john-hello

Stack Outputs
ApiDistributionEndpoint: https://d1vrpoljefctug.cloudfront.net
ServiceEndpoint: https://5clnzj3knc.execute-api.us-east-1.amazonaws.com/john
...
ApiDistributionId: E2X1H9ZQ1B9U0R
```

部署 CloudFront 分发可能需要 20 分钟或更长时间。

1.  在 AWS 控制台中查看堆栈和 CloudFront 分发。

1.  使用以下 `curl` 命令多次调用堆栈输出中显示的端点，以查看缓存结果的性能差异：

```js
$ curl -s -D - -w "Time: %{time_total}" -o /dev/null  https://<see stack output>.cloudfront.net/hello | egrep -i 'X-Cache|Time' 
X-Cache: Miss from cloudfront
Time: 0.324
$ curl ...
X-Cache: Hit from cloudfront
Time: 0.168
$ curl ...
X-Cache: Hit from cloudfront
Time: 0.170
$ curl ...
X-Cache: Miss from cloudfront
Time: 0.319
$ curl ...
X-Cache: Hit from cloudfront
Time: 0.167
```

1.  完成后使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

在 `serverless.yml` 文件中要注意的第一件事是 `endpointType` 设置为 `REGIONAL`。默认情况下，AWS API Gateway 将提供 CloudFront 分发。然而，我们无法访问此分发，也无法利用其所有功能。此外，默认设置不支持多区域部署。因此，我们指定 `REGIONAL` 以便我们可以自行管理 CDN。接下来，我们需要配置 `Origins`。我们指定 `DomainName` 指向区域 API Gateway 端点，并指定阶段为 `OriginPath`，这样我们就不需要在 URL 中包含它了。

接下来，我们配置 `DefaultCacheBehavior`。我们允许读取和写入方法，并缓存读取方法。我们将 `DefaultTTL` 设置为零以确保在服务代码中设置的 `Cache-Control` 头部用于控制 TTL。在本配方中，代码将 `max-age` 设置为 5 秒，我们可以看到我们的缓存命中响应速度大约是两倍。我们还设置 `Compress` 为 `true` 以最小化数据传输，从而提高性能并帮助降低成本。向前转发所有后端期望的 `Headers` 非常重要。例如，授权头部对于使用 OAuth 2.0 保护服务至关重要，正如我们将在第五章 *使用 OAuth 2.0 保护 API 网关* 配方中讨论的那样，*保护云原生系统*。

注意，`ApiGatewayRestApi` 是由 Serverless Framework 创建和控制的逻辑资源 ID。

# 从 CDN 提供静态 JSON

在 *实现搜索 BFF* 配方中，我们创建了一个服务，通过 API Gateway 从 Elasticsearch 中的物化视图中提供一些内容，并直接从 S3 中的物化视图中提供其他内容。这是一个非常棒的方法，可以在极端高负载下以成本效益的方式交付。在本配方中，我们将演示如何为 API Gateway 和 S3 前面添加单个 CloudFront 分发，以将这些内部设计决策封装在单个域名之后。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch4/cdn-json --path cncb-cdn-json
```

1.  使用 `cd cncb-cdn-json` 切换到 `cncb-cdn-json` 目录。

1.  查看以下内容的 `serverless.yml` 文件：

```js
service: cncb-cdn-json

provider:
  name: aws
  runtime: nodejs8.10
  endpointType: REGIONAL
  ...

functions:
  search:
    handler: handler.search
    ...

resources:
  Resources:
    ApiDistribution:
      Type: AWS::CloudFront::Distribution
      Properties:
        DistributionConfig:
          ...
          Origins:
            - Id: S3Origin
              DomainName:
                Fn::Join:
                  - "."
                  - - Ref: Bucket
                    - s3.amazonaws.com
              ...
            - Id: ApiGateway
              DomainName:
                Fn::Join:
                  - "."
                  - - Ref: ApiGatewayRestApi
                    - execute-api.${opt:region}.amazonaws.com
              OriginPath: /${opt:stage}
              ...
          ...
          DefaultCacheBehavior:
            TargetOriginId: S3Origin
            AllowedMethods:
              - GET
              - HEAD
            CachedMethods:
              - HEAD
              - GET
            ...
            MinTTL: 0
            DefaultTTL: 0
            ...
          CacheBehaviors:
            - TargetOriginId: ApiGateway
              PathPattern: /search
              AllowedMethods: [ GET, HEAD, OPTIONS ]
              CachedMethods: [ GET, HEAD, OPTIONS ]
              ...
              MinTTL: 0
              DefaultTTL: 0
              ...

    Bucket:
      Type: AWS::S3::Bucket
...
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cdn-json@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cdn-json
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
endpoints:
  GET - https://hvk3o94wij.execute-api.us-east-1.amazonaws.com/john/search
functions:
  search: cncb-cdn-json-john-search
  load: cncb-cdn-json-john-load

Stack Outputs
...
ApiDistributionEndpoint: https://dfktdq2w7ht2p.cloudfront.net
BucketName: cncb-cdn-json-john-bucket-ls21fzjp2qs2
ServiceEndpoint: https://hvk3o94wij.execute-api.us-east-1.amazonaws.com/john
ApiDistributionId: E3JJREB1B4TIGL
```

部署 CloudFront 分发可能需要 20 分钟或更长时间。

1.  在 AWS 控制台中查看堆栈和 CloudFront 分发。

1.  使用以下命令加载数据：

```js
$ sls invoke -f load -r us-east-1 -s $MY_STAGE -d '{"id":"44444444-5555-1111-1111-000000000000","name":"thing one"}'

{
    "ETag": "\"3f4ca01316d6f88052a940ab198b2dc7\""
}

$ sls invoke -f load -r us-east-1 -s $MY_STAGE -d '{"id":"44444444-5555-1111-2222-000000000000","name":"thing two"}'

{
    "ETag": "\"926f41091e2f47208f90f9b9848dffd0\""
}

$ sls invoke -f load -r us-east-1 -s $MY_STAGE -d '{"id":"44444444-5555-1111-3333-000000000000","name":"thing three"}'

{
    "ETag": "\"2763ac570ff0969bd42182506ba24dfa\""
}
```

1.  使用以下 `curl` 命令调用堆栈输出中显示的端点：

```js
$ curl https://<see stack output>.cloudfront.net/search | json_pp 
[
   "https://dfktdq2w7ht2p.cloudfront.net/things/44444444-5555-1111-1111-000000000000",
   "https://dfktdq2w7ht2p.cloudfront.net/things/44444444-5555-1111-2222-000000000000",
   "https://dfktdq2w7ht2p.cloudfront.net/things/44444444-5555-1111-3333-000000000000"
] $ curl -s -D - -w "Time: %{time_total}" -o /dev/null https://<see stack output>.cloudfront.net/things/44444444-5555-1111-1111-000000000000 | egrep -i 'X-Cache|Time' 
X-Cache: Miss from cloudfront
Time: 0.576
$ curl ...
X-Cache: Hit from cloudfront
Time: 0.248
```

1.  使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

此配方基于 *在 CDN 后部署服务* 配方。本配方中的主要区别在于我们有多达多个 `Origins` 和多个 `CacheBehaviors`，每个分别对应我们的 `Bucket` 和 `ApiGatewayRestApi`。我们为我们的 `S3Origin` 使用 `DefaultCacheBehavior`，因为我们可以在桶中存储许多不同的业务域，路径不同。相反，有一个单一的 `PathPattern` (`/search`) 需要指向我们的 `APIGateway` 原点，因此我们在 `CacheBehaviors` 下定义它。再次强调，在所有情况下，我们将 `DefaultTTL` 设置为零以确保我们的缓存控制头控制 TTL。最终结果是，我们的多个原点现在从外部看起来像一个。

注意，`ApiGatewayRestApi` 是由 Serverless Framework 创建和控制的逻辑资源 ID。

# 触发 CDN 中内容的失效

在 *实现搜索 BFF* 配方中，我们从 S3 静态服务动态 JSON 内容，在 *从 CDN 服务器静态 JSON* 配方中，我们添加了一个具有长最大年龄的缓存控制头，以进一步提高性能。这种技术对于动态且变化缓慢且不可预测的内容非常有效。在此配方中，我们将展示如何通过响应数据更改事件并使缓存失效，以便检索最新数据。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch4/cdn-invalidate --path cncb-cdn-invalidate
```

1.  使用 `cd cncb-cdn-invalidate` 命令导航到 `cncb-cdn-invalidate` 目录。

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-cdn-invalidate

provider:
  name: aws
  runtime: nodejs8.10
  ...

functions:
  load:
    ...
  trigger:
    handler: handler.trigger
    events:
      - sns:
          arn: 
            Ref: BucketTopic
          topicName: ${self:service}-${opt:stage}-trigger
    environment:
      DISABLED: false
      DISTRIBUTION_ID:
        Ref: ApiDistribution

resources:
  Resources:
    Bucket:
      Type: AWS::S3::Bucket
      DependsOn: [ BucketTopic, BucketTopicPolicy ]
      Properties:
        NotificationConfiguration:
          TopicConfigurations:
            - Event: s3:ObjectCreated:Put
              Topic:
                Ref: BucketTopic
    BucketTopic: 
      Type: AWS::SNS::Topic
      Properties: 
        TopicName: ${self:service}-${opt:stage}-trigger
    BucketTopicPolicy: ${file(includes.yml):BucketTopicPolicy}

    ApiDistribution:
      Type: AWS::CloudFront::Distribution
      Properties:
        DistributionConfig:
          ...
```

1.  查看名为 `handler.js` 的文件，其内容如下：

```js
module.exports.trigger = (event, context, cb) => {
  _(process.env.DISABLED === 'true' ? [] : event.Records)
    .flatMap(messagesToTriggers)
    .flatMap(invalidate)
    .collect()
    .toCallback(cb);
};

const messagesToTriggers = r => _(JSON.parse(r.Sns.Message).Records);

const invalidate = (trigger) => {
  const params = {
    DistributionId: process.env.DISTRIBUTION_ID,
    InvalidationBatch: {
      CallerReference: uuid.v1(),
      Paths: {
        Quantity: 1,
        Items: [`/${trigger.s3.object.key}`]
      }
    }
  };

  const cf = new aws.CloudFront();
  return _(cf.createInvalidation(params).promise());
};
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cdn-invalidate@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cdn-invalidate
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
functions:
  load: cncb-cdn-invalidate-john-load
  trigger: cncb-cdn-invalidate-john-trigger

Stack Outputs
BucketArn: arn:aws:s3:::cncb-cdn-invalidate-john-bucket-z3u7jc60piub
BucketName: cncb-cdn-invalidate-john-bucket-z3u7jc60piub
ApiDistributionEndpoint: https://dgmob5bgqpnpi.cloudfront.net
TopicArn: arn:aws:sns:us-east-1:123456789012:cncb-cdn-invalidate-john-trigger
...
ApiDistributionId: EV1QUKWEQV6XN
```

部署 CloudFront 分发可能需要 20 分钟或更长时间。

1.  在 AWS 控制台中查看堆栈。

1.  使用以下命令加载数据：

```js
$ sls invoke -f load -r us-east-1 -s $MY_STAGE -d '{"id":"44444444-6666-1111-1111-000000000000","name":"thing one"}'

{
    "ETag": "\"3f4ca01316d6f88052a940ab198b2dc7\""
}
```

1.  使用以下 `curl` 命令调用堆栈输出中的端点：

```js
$ curl https://<see stack output>.cloudfront.net/things/44444444-6666-1111-1111-000000000000 | json_pp 
{
   "name" : "thing one",
   "id" : "44444444-6666-1111-1111-000000000000"
}
```

1.  使用以下命令加载数据：

```js
$ sls invoke -f load -r us-east-1 -s $MY_STAGE -d '{"id":"44444444-6666-1111-1111-000000000000","name":"thing one again"}'

{
    "ETag": "\"edf9676ddcc150f722f0f74b7a41bd7f\""
}
```

1.  查看 `trigger` 函数的日志：

```js
$ sls logs -f trigger -r us-east-1 -s $MY_STAGE

2018-05-20 02:43:16 ... params: {"DistributionId":"EV1QUKWEQV6XN","InvalidationBatch":{"CallerReference":"13a8e420-5bf9-11e8-818d-9de0acd70c96","Paths":{"Quantity":1,"Items":["/things/44444444-6666-1111-1111-000000000000"]}}}
2018-05-20 02:43:17 ... {"Location":"https://cloudfront.amazonaws.com/2017-10-30/distribution/EV1QUKWEQV6XN/invalidation/I33URVVAO02X7I","Invalidation":{"Id":"I33URVVAO02X7I","Status":"InProgress","CreateTime":"2018-05-20T06:43:17.102Z","InvalidationBatch":{"Paths":{"Quantity":1,"Items":["/things/44444444-6666-1111-1111-000000000000"]},"CallerReference":"13a8e420-5bf9-11e8-818d-9de0acd70c96"}}}
```

1.  使用以下 `curl` 命令调用堆栈输出中的端点，直到失效完成且输出更改：

```js
$ curl https://<see stack output>.cloudfront.net/things/44444444-6666-1111-1111-000000000000 | json_pp 
{
   "name" : "thing one again",
   "id" : "44444444-6666-1111-1111-000000000000"
}
```

1.  完成后，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

此配方基于 *在 S3 中实现物化视图* 和 *实现搜索 BFF* 配方。我们定义存储桶的 `NotificationConfiguration` 以将事件发送到 SNS `BucketTopic`，以便我们可以触发多个消费者。在 *实现搜索 BFF* 配方中，我们在 Elasticsearch 中触发了索引。在此配方中，我们展示了如何触发 CloudFront 分发中缓存的失效。`trigger` 函数为每个 `trigger.s3.object.key` 创建一个失效请求。这些失效将迫使 CDN 在下一次请求时从源检索这些资源。

缓存失效的缓慢流程不会产生显著成本。然而，大量单个失效批次可能会产生费用。这可能在增强服务时的数据转换过程中发生。在这些情况下，在执行转换之前应将 `DISABLED` 环境变量设置为 `true`。然后，在转换完成后，使用通配符手动失效分发。

# 在云边缘执行代码

AWS Lambda@Edge 是一个功能，允许函数在云边缘响应 CloudFront 事件时执行。这种能力为在云边缘以极低延迟控制、修改和生成内容提供了有趣的机会。我认为我们才刚刚开始挖掘可以使用这个新功能实现的应用场景。这个配方演示了将裸骨函数与 CloudFront 分发关联起来。当请求中不存在授权头时，该函数会以未授权（403）状态码响应。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch4/cdn-lambda --path cncb-cdn-lambda
```

1.  使用 `cd cncb-cdn-lambda` 命令导航到 `cncb-cdn-lambda` 目录。

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-cdn-lambda

plugins:
  - serverless-plugin-cloudfront-lambda-edge

provider:
  name: aws
  runtime: nodejs8.10
  ...

functions:
  authorize:
    handler: handler.authorize
    memorySize: 128
 timeout: 1
 lambdaAtEdge:
 distribution: 'ApiDistribution'
 eventType: 'viewer-request'
  ...

resources:
  Resources:
    Bucket:
      Type: AWS::S3::Bucket

    ApiDistribution:
      Type: AWS::CloudFront::Distribution
      Properties:
        DistributionConfig:
          ...
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容：

```js
...
"LambdaFunctionAssociations": [
  {
    "EventType": "viewer-request",
    "LambdaFunctionARN": {
      "Ref": "AuthorizeLambdaVersionSticfT7s2DCStJsDgzhTCrXZB1CjlEXXc3bR4YS1owM"
    }
  }
]
...
```

1.  部署栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cdn-lambda@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cdn-lambda
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
functions:
  authorize: cncb-cdn-lambda-john-authorize
  load: cncb-cdn-lambda-john-load

Stack Outputs
ApiDistributionEndpoint: https://d1kktmeew2xtn2.cloudfront.net
BucketName: cncb-cdn-lambda-john-bucket-olzhip1qvzqi
...
ApiDistributionId: E2VL0VWTW5IXUA

```

部署 CloudFront 分发可能需要 20 分钟或更长时间。对这些函数的任何更改都将导致分发的重新部署，因为分发与函数版本相关联。

1.  在 AWS 控制台中查看栈、函数和 CloudFront 分发。

1.  使用以下命令加载数据：

```js
$ sls invoke -f load -r us-east-1 -s $MY_STAGE -d '{"id":"44444444-7777-1111-1111-000000000000","name":"thing one"}'

{
    "ETag": "\"3f4ca01316d6f88052a940ab198b2dc7\""
}
```

1.  使用以下 `curl` 命令调用栈输出中显示的端点：

```js
$ curl -v -H "Authorization: Bearer 1234567890" https://<see stack output>.cloudfront.net/things/44444444-7777-1111-1111-000000000000 | json_pp 
...
> Authorization: Bearer 1234567890
>
< HTTP/1.1 200 OK
...
{
   "name" : "thing one",
   "id" : "44444444-7777-1111-1111-000000000000"
}$ curl -v https://<see stack output>.cloudfront.net/things/44444444-7777-1111-1111-000000000000 | json_pp
...
< HTTP/1.1 401 Unauthorized
...
[
   {
      "Records" : [
         {
            "cf" : {
               "config" : {
                  "eventType" : "viewer-request",
                  "requestId" : "zDpj1UckJLTIln8dwak2ZL1SX0LtWfGPhg3mC1EGIgfRd6gzJFVeqg==",
                  "distributionId" : "E2VL0VWTW5IXUA",
                  "distributionDomainName" : "d1kktmeew2xtn2.cloudfront.net"
               },
           ...
            }
         }
      ]
   },
   {
    ...
      "logGroupName" : "/aws/lambda/us-east-1.cncb-cdn-lambda-john-authorize",
    ...
   },
   {
      "AWS_REGION" : "us-east-1",
    ...
   }
]
```

1.  在 AWS 控制台中查看日志。

1.  使用 `npm run rm:lcl -- -s $MY_STAGE` 命令完成后删除栈。

Lambda@Edge 根据分发的 `PriceClass` 将函数复制到多个区域。在所有副本被删除之前，无法删除函数。这应该在删除分发后的几小时内发生。因此，删除栈最初会失败，可以在副本被删除后重复执行。

# 它是如何工作的...

在这个配方中，我们使用 `serverless-plugin-cloudfront-lambda-edge` 插件将 `authorize` 函数与 CloudFront 的 `ApiDistribution` 关联起来。我们在 `lambdaAtEdge` 元素下指定了 `distribution` 和 `eventType`。然后插件使用这些信息在生成的 CloudFormation 模板中的分发上创建 `LambdaFunctionAssociations` 元素。`eventType` 可以设置为 `viewer-request`、`origin-request`、`origin-response` 或 `view-response`。这个配方使用 `viewer-request`，因为它需要访问由观众发送的授权头。我们明确设置了函数的 `memorySize` 和 `timeout`，因为 Lambda@Edge 对这些值有限制。

Lambda@Edge 在与特定边缘位置关联的区域记录 `console.log` 语句。这个配方在响应中返回 `logGroupName` 和 `AWS_REGION`，以帮助演示如何找到日志。
