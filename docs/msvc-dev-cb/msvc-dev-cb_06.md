# 第六章：安全性

在本章中，我们将介绍以下菜谱：

+   验证您的微服务

+   保护容器

+   安全配置

+   安全日志

+   基础设施即代码

# 简介

与本书中涵盖的许多主题一样，微服务架构中的安全性是关于权衡的。在微服务架构中，单个代码库的责任有限。如果攻击者能够破坏单个运行的服务，他们只能执行由该特定微服务管理的操作。然而，微服务架构的分布式特性意味着攻击者有更多的目标可以在运行在不同集群中的服务中进行潜在利用。这些集群之间的网络流量，包括边缘服务和内部服务之间的流量，为攻击者提供了许多发现漏洞的机会。

由于微服务架构的分布式特性，在配置服务如何相互通信时必须考虑网络拓扑。这种关注也存在于单体代码库中，其中单个代码库的运行实例需要通过网络与数据库服务器、缓存、负载均衡器等进行通信。可以认为微服务架构使这些挑战更加明显，因此迫使工程师更早地考虑这些问题。

安全性是一个大话题。本章将讨论在构建、部署和运营微服务时需要考虑的一些良好实践，但重要的是要注意，这并不是一个详尽的考虑清单。在开发任何系统时，都应考虑良好的 API 实践和深度防御，微服务也不例外。我强烈推荐**OWASP** ([`www.owasp.org/index.php/Main_Page`](https://www.owasp.org/index.php/Main_Page)) 作为学习更多关于 Web 应用安全资源的参考。

# 验证您的微服务

在第一章，*打破单体*，我们介绍了一个 Ruby on Rails 代码库，它为我们的虚构图片分享应用`pichat`提供动力。Rails 代码库通过检查授权头来验证每个请求。如果存在头信息，应用程序将尝试使用从环境变量中读取的共享密钥对其进行解码（参见*安全*配置菜谱）。如果授权头中提供的令牌有效，解码的值包含有关用户的环境信息，包括用户 ID。然后使用这些信息从数据库中检索用户，以便应用程序对发出请求的用户有上下文。如果授权头缺失或无法成功解码，应用程序将引发异常，并向调用者返回 HTTP 401，包括错误信息。为了获取要包含在授权头中的令牌，客户端应用程序可以向`/auth/login`端点发送带有有效用户凭据的`POST`请求。以下 CURL 命令演示了此流程：

```js
$ curl -D - -X POST http://localhost:9292/auth/login -d'email=p@eval.ca&password=foobar123'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
ETag: W/"3675d2006d59e01f8665f20ffef65fe7"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: 6660a102-059f-4afe-b17c-99375db305dd
X-Runtime: 0.150903
Transfer-Encoding: chunked

{"auth_token":"eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJleHAiOjE1MzE2ODUxNjR9.vAToW_mWlOnr-GPzP79EvN62Q2MpsnLIYanz3MTbZ5Q"}
```

现在我们有了令牌，我们可以将其包含在后续请求的头部中：

```js
$ curl -X POST -D - -H 'Authorization: eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJleHAiOjE1MzE2ODUxNjR9.vAToW_mWlOnr-GPzP79EvN62Q2MpsnLIYanz3MTbZ5Q' http://localhost:9292/messages -d'body=Hello&user_id=1'
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
ETag: W/"211cdab551e63ca48de48217357f1cf7"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: 1525333c-dada-40ff-8c25-a0e7d151433c
X-Runtime: 0.019609
Transfer-Encoding: chunked

{"id":1,"body":"Hello","user_id":1,"created_at":"2018-07-14T20:08:19.369Z","updated_at":"2018-07-14T20:08:19.369Z","from_user_id":1}
```

由于`pichat-api`是一个单体代码库，它正在扮演许多不同的角色以支持此流程。它正在充当授权服务、认证网关、用户存储和授权客户端。这种责任的耦合正是我们希望在微服务架构中避免的。

幸运的是，在保持流程不变的同时，很容易将这些责任分割到单独的代码库中。使用共享密钥在**JSON Web Tokens**（**JWT**）中编码信息允许单个微服务在不需要对每个请求都向集中式认证服务发出请求的情况下安全地验证请求。获取认证令牌可以是集中式服务的责任，但可以使用 API 网关或前端的后端使这一事实对客户端透明。以下图表演示了某些责任将如何分割：

![图片](img/995d3ee1-7c63-4015-85ea-eafd43997426.png)

我们将创建一个**认证服务**来处理用户注册并交换 JWT 凭证。然后我们将使用我们在第二章，*边缘服务*中介绍的 Zuul 开源项目创建一个简单的**API 网关**。

# 如何操作...

让我们看看以下步骤：

1.  让我们创建一个认证服务。使用以下`build.gradle`文件创建一个新的 Java 项目：

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
apply plugin: 'io.spring.dependency-management'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web'
    compile group: 'org.springframework.security', name: 'spring-security-core'
    compile group: 'org.springframework.security', name: 'spring-security-config'
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-jpa'
```

```js
    compile group: 'io.jsonwebtoken', name: 'jjwt', version: '0.9.1'
    compile group: 'mysql', name: 'mysql-connector-java'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

我们将在 MySQL 数据库中存储用户凭证，因此我们声明`mysql-connector-java`作为依赖项。我们还将使用一个名为`jjwt`的开源 JWT 库。

存储用户凭据是一个重要的话题。用户密码绝不应该以纯文本形式存储，许多散列算法，如 MD5 和 SHA1，已被证明容易受到各种暴力攻击。在这个例子中，我们将使用 `bcrypt`。在实际应用中，我们会考虑多个散列步骤，例如首先使用 SHA512 散列，然后通过 `bcrypt`。我们还会考虑为每个用户添加一个盐。**开放网络应用安全项目**有很多关于存储密码的优秀建议：[`www.owasp.org/index.php/Password_Storage_Cheat_Sheet`](https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet)。

1.  创建一个名为 `Application` 的新类。它将包含我们的主方法以及 `PasswordEncoder`：

```js
package com.packtpub.microservices.ch06.auth;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@SpringBootApplication
public class Application {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

1.  我们将用户凭据建模为一个简单的 POJO，包含 `email` 和 `password` 字段。创建一个名为 `com.packtpub.microservices.ch06.auth.models` 的新包和一个名为 `UserCredential` 的新类：

```js
package com.packtpub.microservices.ch06.auth.models;

import org.hibernate.annotations.GenericGenerator;

import javax.persistence.*;

@Entity
public class UserCredential {
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String id;

    @Column(unique=true)
    private String email;

    private String password;

    public UserCredential(String email) {
        this.email = email;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

```

```js
    public void setPassword(String password) {
        this.password = password;
    }
}
```

1.  创建一个模型来表示对成功登录和注册请求的响应。成功的响应将包含一个包含 JWT 的 JSON 文档。创建一个名为 `AuthenticationToken` 的新类：

```js
package com.packtpub.microservices.ch06.auth.models;

import com.fasterxml.jackson.annotation.JsonProperty;

public class AuthenticationToken {

    @JsonProperty("auth_token")
    private String authToken;

    public AuthenticationToken() {}

    public AuthenticationToken(String authToken) {
        this.authToken = authToken;
    }

    public String getAuthToken() {
        return this.authToken;
    }

    public void setAuthToken(String authToken) {
        this.authToken = authToken;
    }
}
```

1.  `UserCredential` 类将通过 Java 持久化 API 访问。为此，我们必须首先创建 `CrudRepository`。创建一个名为 `com.packtpub.microservices.ch06.auth.data` 的新包和一个名为 `UserCredentialRepository` 的新类。除了继承自 `CrudRepository`，我们还将定义一个用于通过电子邮件检索 `UserCredential` 实例的单个方法：

```js
package com.packtpub.microservices.ch06.auth.data;

import com.packtpub.microservices.ch06.auth.models.UserCredential;
import org.springframework.data.repository.CrudRepository;

public interface UserCredentialRepository extends CrudRepository<UserCredential, String> {
    UserCredential findByEmail(String email);
}
```

1.  当用户尝试使用无效凭据注册或登录时，我们希望返回 HTTP 401 状态码以及一条消息，指出他们提供了无效凭据。为了做到这一点，我们将在我们的控制器方法中创建一个单独的异常：

```js
package com.packtpub.microservices.ch06.auth.exceptions;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.UNAUTHORIZED)
public class InvalidCredentialsException extends Exception {
    public InvalidCredentialsException(String message) { super(message); }
}
```

1.  创建控制器。登录和注册端点将由单个控制器提供服务。注册方法将简单地验证输入并创建一个新的 `UserCredential` 实例，使用我们之前创建的 `CrudRepository` 包持久化它。然后，它将使用新注册用户的用户 ID 作为主题编码 JWT。登录方法将验证提供的凭据并提供一个以用户 ID 作为主题的 JWT。控制器需要访问在主类中定义的 `UserCredentialRepository` 和 `PasswordEncoder`。创建一个名为 `com.packtpub.microservices.ch06.auth.controllers` 的新包和一个名为 `UserCredentialController` 的新类：

```js
package com.packtpub.microservices.ch06.auth.controllers;

import com.packtpub.microservices.ch06.auth.data.UserCredentialRepository;
import com.packtpub.microservices.ch06.auth.exceptions.InvalidCredentialsException;
import com.packtpub.microservices.ch06.auth.models.AuthenticationToken;
import com.packtpub.microservices.ch06.auth.models.UserCredential;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

import javax.crypto.spec.SecretKeySpec;
import javax.xml.bind.DatatypeConverter;
import java.security.Key;

@RestController
public class UserCredentialController {

    @Autowired
    private UserCredentialRepository userCredentialRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Value("${secretKey}")
    private String keyString;

    private String encodeJwt(String userId) {
        System.out.println("SIGNING KEY: " + keyString);
        Key key = new SecretKeySpec(
                DatatypeConverter.parseBase64Binary(keyString),
                SignatureAlgorithm.HS256.getJcaName());

        JwtBuilder builder = Jwts.builder().setId(userId)
                .setSubject(userId)
                .setIssuer("authentication-service")
                .signWith(SignatureAlgorithm.HS256, key);

        return builder.compact();
    }

    @RequestMapping(path = "/register", method = RequestMethod.POST, produces = "application/json")
    public AuthenticationToken register(@RequestParam String email, @RequestParam String password, @RequestParam String passwordConfirmation) throws InvalidCredentialsException {
        if (!password.equals(passwordConfirmation)) {
            throw new InvalidCredentialsException("Password and confirmation do not match");
        }

        UserCredential cred = new UserCredential(email);
        cred.setPassword(passwordEncoder.encode(password));
        userCredentialRepository.save(cred);

        String jws = encodeJwt(cred.getId());
        return new AuthenticationToken(jws);
    }

    @RequestMapping(path = "/login", method = RequestMethod.POST, produces = "application/json")
    public AuthenticationToken login(@RequestParam String email, @RequestParam String password) throws InvalidCredentialsException {
        UserCredential user = userCredentialRepository.findByEmail(email);

        if (user == null || !passwordEncoder.matches(password, user.getPassword())) {
            throw new InvalidCredentialsException("Username or password invalid");
        }

        String jws = encodeJwt(user.getId());
        return new AuthenticationToken(jws);
    }
}
```

1.  由于我们正在连接到本地数据库，并且我们在签名 JWT 时使用共享密钥，因此我们需要创建一个小的属性文件。在 `src/main/resources` 目录下创建一个名为 `application.yml` 的文件：

```js
server:
  port: 8081

spring:
  jpa.hibernate.ddl-auto: create
  datasource.url: jdbc:mysql://localhost:3306/user_credentials
  datasource.username: root
  datasource.password:

secretKey: supers3cr3t
```

现在我们有一个工作的认证服务，下一步是创建一个简单的 API 网关，使用开源网关服务 Zuul。除了将请求路由到下游服务外，API 网关还将使用认证过滤器来验证需要认证的请求中是否传递了有效的 JWT。

1.  创建一个具有以下`build.gradle`文件的 Java 项目：

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
apply plugin: 'io.spring.dependency-management'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencyManagement {
    imports {
        mavenBom 'org.springframework.cloud:spring-cloud-netflix:1.4.4.RELEASE'
    }
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web'
    compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-zuul'
    compile group: 'org.springframework.security', name: 'spring-security-core'
    compile group: 'org.springframework.security', name: 'spring-security-config'
    compile group: 'org.springframework.security', name: 'spring-security-web'
    compile group: 'io.jsonwebtoken', name: 'jjwt', version: '0.9.1'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

注意，我们正在使用与认证服务相同的 JWT 库。

1.  创建一个名为`com.packtpub.microservices.ch06.gateway`的新包和一个名为`Application`的新类：

```js
package com.packtpub.microservices.ch06.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

1.  我们将通过创建`OncePerRequestFilter`的子类来创建一个认证过滤器，该过滤器旨在为每个请求调度提供单次执行。该过滤器将解析`Authorization`头中的 JWT，并尝试使用共享密钥对其进行解码。如果 JWT 可以被验证和解码，我们可以确信它是由有权访问共享密钥的发行者编码的。我们将此视为我们的信任边界；任何有权访问共享密钥的人都可以被信任，因此我们可以信任 JWT 的主题是认证用户的 ID。创建一个名为`AuthenticationFilter`的新类：

```js
package com.packtpub.microservices.ch06.gateway;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.xml.bind.DatatypeConverter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Optional;

public class AuthenticationFilter extends OncePerRequestFilter {

    private String signingSecret;

    AuthenticationFilter(String signingSecret) {
        this.signingSecret = signingSecret;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        Optional<String> token = Optional.ofNullable(request.getHeader("Authorization"));
        Optional<Authentication> auth = token.filter(t -> t.startsWith("Bearer")).flatMap(this::authentication);
        auth.ifPresent(a -> SecurityContextHolder.getContext().setAuthentication(a));
        filterChain.doFilter(request, response);
    }

    private Optional<Authentication> authentication(String t) {
        System.out.println(signingSecret);
        String actualToken = t.substring("Bearer ".length());
        try {
            Claims claims = Jwts.parser()
                    .setSigningKey(DatatypeConverter.parseBase64Binary(signingSecret))
                    .parseClaimsJws(actualToken).getBody();
            Optional<String> userId = Optional.ofNullable(claims.getSubject()).map(Object::toString);
            return userId.map(u -> new UsernamePasswordAuthenticationToken(u, null, new ArrayList<SimpleGrantedAuthority>()));
        } catch (Exception e) {
            return Optional.empty();
        }

    }
}
```

1.  将此与 API 网关项目的安全配置一起连接起来。创建一个名为`SecurityConfig`的新类：

```js
package com.packtpub.microservices.ch06.gateway;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import javax.servlet.http.HttpServletResponse;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Value("${jwt.secret}")
    private String signingSecret;

    @Override
    protected void configure(HttpSecurity security) throws Exception {
        security
            .csrf().disable()
            .logout().disable()
            .formLogin().disable()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
                .anonymous()
            .and()
                .exceptionHandling().authenticationEntryPoint(
                    (req, rsp, e) -> rsp.sendError(HttpServletResponse.SC_UNAUTHORIZED))
            .and()
            .addFilterAfter(new AuthenticationFilter(signingSecret),
                    UsernamePasswordAuthenticationFilter.class)
            .authorizeRequests()
            .antMatchers("/auth/**").permitAll()
            .antMatchers("/messages/**").authenticated()
            .antMatchers("/users/**").authenticated();
    }
}

```

如我们所见，我们允许对认证服务（以`/auth/...`为前缀的请求）的任何请求。我们要求对用户或消息服务的请求进行认证。

1.  我们需要一个配置文件来存储共享密钥以及 Zuul 服务器的路由信息。在`src/main/resources`目录中创建一个名为`application.yml`的文件：

```js
server:
  port: 8080

jwt:
  secret: supers3cr3t

zuul:
  routes:
    authentication-service:
      path: /auth/**
      url: http://127.0.0.1:8081
    message-service:
      path: /messages/**
      url: http://127.0.0.1:8082
    user-service:
      path: /users/**
      url: http://127.0.0.1:8083
```

1.  现在我们有一个工作认证服务和能够验证 JWT 的 API 网关，我们可以通过使用前面配置文件中定义的端口来运行 API 网关、认证服务和消息服务来测试我们的认证方案。以下 CURL 请求现在显示，有效的凭据可以交换 JWT，并且 JWT 可以用来访问受保护资源。我们还可以显示，没有有效 JWT 的请求会被拒绝访问受保护资源。

注意，在这个例子中，消息服务仍然不做任何请求授权。任何进行认证请求的人理论上都可以访问任何人的消息。消息服务应该修改为检查 JWT 主题中的用户 ID，并且只允许访问属于该用户的消息。

1.  我们可以使用`curl`来测试注册新用户账户：

```js
$ curl -X POST -D - http://localhost:8080/auth/register -d'email=p@eval.ca&password=foobar123&passwordConfirmation=foobar123'

HTTP/1.1 200
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
X-Application-Context: application:8080
Date: Mon, 16 Jul 2018 03:27:17 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked

{"auth_token":"eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJmYWQzMGZiMi03MzhmLTRiM2QtYTIyZC0zZGNmN2NmNGQ1NGIiLCJzdWIiOiJmYWQzMGZiMi03MzhmLTRiM2QtYTIyZC0zZGNmN2NmNGQ1NGIiLCJpc3MiOiJhdXRoZW50aWNhdGlvbi1zZXJ2aWNlIn0.TzOKItjBU-AtRMqIB_D1n-qv6IO_zCBIK8ksGzsTC90"}
```

1.  现在我们有了 JWT，我们可以将其包含在消息服务的请求头中，以测试 API 网关是否能够验证和解码令牌：

```js
$ curl -D - -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI3YmU4N2U3Mi03ZjhhLTQ3ZjktODk3NS1mYzM5ZTE0NjNmODAiLCJzdWIiOiI3YmU4N2U3Mi03ZjhhLTQ3ZjktODk3NS1mYzM5ZTE0NjNmODAiLCJpc3MiOiJhdXRoZW50aWNhdGlvbi1zZXJ2aWNlIn0.fpFbHhdSEVKk95m5Q7iNjkKyM-eHkCGGKchTTKgbGWw" http://localhost:8080/messages/123

HTTP/1.1 404
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
X-Application-Context: application:8080
Date: Mon, 16 Jul 2018 04:05:40 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked

{"timestamp":1532318740403,"status":404,"error":"Not Found","exception":"com.packtpub.microservices.ch06.message.exceptions.MessageNotFoundException","message":"Message 123 could not be found","path":"/123"}
```

我们从消息服务得到 404 错误表明请求已经到达该服务。如果我们修改请求头中的 JWT，我们应该得到 401 错误：

```js
$ curl -D - -H "Authorization: Bearer not-the-right-jwt" http://localhost:8080/messages/123

HTTP/1.1 401
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Mon, 23 Jul 2018 04:06:47 GMT

{"timestamp":1532318807874,"status":401,"error":"Unauthorized","message":"No message available","path":"/messages/123"}
```

# 保护容器

容器的出现为管理微服务架构的组织解决了许多问题。容器允许将服务捆绑为自包含的单元，软件及其依赖项可以构建为一个单一的项目，然后将其运送到任何环境中运行或调度。而不是依赖于复杂的配置管理解决方案来管理生产系统的微小更改，容器支持不可变基础设施的概念；一旦基础设施构建完成，就不需要升级或维护。相反，你只需构建新的基础设施，然后丢弃旧的。

容器还允许组织优化其存储和计算资源的使用。由于软件可以构建为容器，多个应用程序可以在单个虚拟机或硬件上运行，每个应用程序都意识不到其他应用程序的存在。虽然多租户有许多优点，但在同一虚拟机上运行多个服务引入了新的攻击场景，恶意用户可能会利用这些场景。如果攻击者能够利用一个服务中的漏洞，他们可能会利用这个漏洞攻击同一虚拟机上运行的其他服务。在这种配置中，默认情况下，集群被视为安全边界；如果你有权访问集群，你必须被信任。

根据组织的需要，将集群视为安全边界可能不足以满足需求，并且可能希望容器之间有更多的安全和隔离。seccomp 安全设施在 Linux 内核的 2.6.12 版本中被引入。它支持限制从进程可以做出的系统调用。使用 seccomp 策略运行容器化应用程序实际上将服务以及容器中运行的任何其他进程隔离。在这个菜谱中，我们将向您展示如何检查 Linux 内核中是否配置了 seccomp，并演示如何使用自定义 seccomp 策略运行容器。

# 如何操作...

1.  为了在 Docker 容器中使用 seccomp 策略，你必须在配置了 seccomp 支持的 Linux 内核的主机操作系统上运行容器。要检查这一点，你可以在内核配置文件中搜索 `CONFIG_SECCOMP`：

```js
$ grep CONFIG_SECCOMP= /boot/config-$(uname -r)
CONFIG_SECCOMP=y
```

1.  现在我们已经验证了 Linux 内核中已启用 seccomp，我们可以查看 Docker 一起打包的默认配置文件（[`github.com/moby/moby/blob/master/profiles/seccomp/default.json`](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)）。此默认策略对于大多数需求来说已经足够，并且相当严格。如果启用了 seccomp 支持，容器将使用此策略运行。

1.  为了进一步验证 seccomp 是否已配置并且 Docker 能够支持它，我们将创建一个简单的自定义策略，然后在容器中运行一个命令来演示策略正在被强制执行。创建一个名为 `policy.json` 的文件：

```js
{
    "defaultAction": "SCMP_ACT_ALLOW",
    "syscalls": [
        {
            "name": "chown",
            "action": "SCMP_ACT_ERRNO"
        }
    ]
}
```

1.  现在，运行一个执行 shell 的容器，然后尝试创建一个文件，然后更改所有权。错误消息表明容器正在受到 seccomp 策略的限制：

```js
$ docker run --rm -it --security-opt seccomp:policy.json busybox /bin/sh
/ # touch foo
/ # chown root foo
chown: foo: Operation not permitted
```

# 安全配置

服务通常需要某种形式的配置。服务配置存储了所有可能根据服务部署的环境而变化的信息。例如，当在开发人员的工作站上本地运行服务时，服务可能应该连接到也在本地运行的数据库。然而，在生产环境中，服务应该连接到生产数据库。配置中通常存储的数据包括数据存储的位置和凭证、访问令牌或其他第三方服务的凭证以及操作信息，例如发送指标的位置或初始化连接池或配置网络连接超时时应使用的值。

将配置与代码分开存储很重要。当你进行配置更改时，你不应该需要将更改提交到源代码存储库、创建新的构建和运行单独的部署。理想情况下，应该有一种简单的方法来更改配置，而无需部署服务的新版本。将配置存储在代码中（例如，在源代码文件中硬编码密码）也是从安全角度的不良做法。任何有权访问源代码的人都可以访问配置，在秘密的情况下，这通常是不希望的。尽可能频繁地滚动密钥和凭证是一种良好的做法，这样即使秘密被泄露或容易受到泄露的威胁，它也不会有效很长时间。硬编码秘密会使这变得困难，在实践中通常意味着它不会发生。

一种常见的最佳实践是将配置存储在环境变量中。这是一种将配置值暴露给进程的好方法，可以根据服务运行的环境轻松更改。环境变量对于非秘密配置值很好，例如主机名、超时和日志级别。环境变量不足以存储秘密。

将秘密存储为环境变量会使这些值对在相同容器或进程空间中运行的所有进程都可用，这使得它们容易受到拦截。有各种方法可以将秘密与应用程序配置的其他部分分开存储。在 Kubernetes 集群上部署的应用程序可以使用一种特殊类型的对象，称为 `secret`，它旨在用于此目的。Kubernetes 机密在节点之间传输时使用驻留在主节点上的私钥进行加密，然而，在静止状态下，机密以明文形式存储。理想情况下，秘密应该以加密值的形式存储，并且只有明确允许这样做的过程才能解密。

Vault 是一个由 HashiCorp 活跃维护的开源项目。其目的是提供一个易于使用的系统，用于安全地存储和访问机密。除了秘密存储外，Vault 还提供访问日志审计、细粒度访问控制和轻松滚动。在这个菜谱中，我们将创建一个新的服务，称为 attachment-service，该服务负责处理消息的图像和视频附件。Attachment-service 将使用 Vault 获取用于上传照片和视频文件的 S3 存储桶的有效 AWS 凭据。该服务还将使用 Vault 获取存储附件元数据的 MySQL 数据库的凭证。非敏感配置，如数据库名称或上传照片和视频的 S3 存储桶名称，将作为环境变量提供给服务。

# 如何做到这一点...

为了演示如何使用 Vault 安全存储敏感配置数据，我们首先将创建一个附件服务，该服务使用环境变量存储敏感信息。然后我们将集成 Vault，以便从安全存储中读取相同的配置：

1.  创建一个名为 `attachment-service` 的新 Java 项目，并使用以下 `build.gradle` 文件：

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
apply plugin: 'io.spring.dependency-management'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web'
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-jpa', version: '1.5.9.RELEASE'
    compile group: 'mysql', name: 'mysql-connector-java'
    compile group: 'com.amazonaws', name: 'aws-java-sdk-s3', version: '1.11.375'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

1.  创建一个名为 `com.packtpub.microservices.ch06.attachment` 的新包，并创建一个名为 `Application` 的新类，该类将作为我们服务的入口点。除了运行我们的 Spring Boot 应用程序外，此类还将公开一个 bean，即 Amazon S3 客户端。请注意，我们目前使用的是 `EnvironmentVariableCredentialsProvider` 类，它从一组环境变量中读取凭证，这不是我们希望在生产中做的事情：

```js
package com.packtpub.microservices.ch06.attachment;

import com.amazonaws.auth.EnvironmentVariableCredentialsProvider;
import com.amazonaws.regions.Regions;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

    @Bean
    public AmazonS3 getS3Client() {
        AmazonS3ClientBuilder client = AmazonS3ClientBuilder.standard();
        return client.withCredentials(
                new EnvironmentVariableCredentialsProvider()).withRegion(Regions.US_WEST_2).build();
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

1.  创建一个名为 `com.packtpub.microservices.ch06.attachment.models` 的新包和一个名为 `Attachment` 的新类。这将是我们存储在关系型数据库中的附件表示：

```js
package com.packtpub.microservices.ch06.attachment.models;

import org.hibernate.annotations.GenericGenerator;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Attachment {

    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String id;

    @Column(unique = true)
    private String messageId;
    private String url;
    private String fileName;
    private Integer mediaType;

    public Attachment(String messageId, String url, String fileName, Integer mediaType) {
        this.messageId = messageId;
        this.url = url;
        this.fileName = fileName;
        this.mediaType = mediaType;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getMessageId() {
        return messageId;
    }

    public void setMessageId(String messageId) {
        this.messageId = messageId;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }

    public Integer getMediaType() {
        return mediaType;
    }

    public void setMediaType(Integer mediaType) {
        this.mediaType = mediaType;
    }
}
```

1.  为了对之前定义的 `Attachment` 类执行基本操作，我们将创建一个名为 `com.packtpub.microservices.ch06.attachment.data` 的新包和一个名为 `AttachmentRepository` 的接口，该接口扩展了 `CrudRepository`。我们还将定义一个自定义方法签名，允许调用者找到与特定消息相关的所有附件：

```js
package com.packtpub.microservices.ch06.attachment.data;

import com.packtpub.microservices.ch06.attachment.models.Attachment;
import org.springframework.data.repository.CrudRepository;

import java.util.List;

public interface AttachmentRepository extends CrudRepository<Attachment, String> {
    public List<Attachment> findByMessageId(String messageId);
}
```

1.  我们还需要一种方式来建模传入的请求。我们的服务将接受作为请求体中 JSON 发送的请求。JSON 对象将包含一个文件名，并包含作为 Base64 编码字符串的文件数据。创建一个名为 `AttachmentRequest` 的新类，其定义如下：

```js
package com.packtpub.microservices.ch06.attachment.models;

import com.fasterxml.jackson.annotation.JsonProperty;

import java.util.Map;

public class AttachmentRequest {
    private String fileName;

    private String data;

    public AttachmentRequest() {}

    public AttachmentRequest(String fileName, String data) {
        this.fileName = fileName;
        this.data = data;
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }

    @JsonProperty("file")
    private void unpackFileName(Map<String, String> file) {
        this.fileName = file.get("name");
        this.data = file.get("data");
    }
}
```

1.  在我们的控制器中，我们将在下一节定义它，如果找不到特定消息的附件，我们需要返回 HTTP 404 响应给调用者。为了做到这一点，创建一个名为 `com.packtpub.microservices.ch06.attachment.exceptions` 的新包和一个名为 `AttachmentNotFoundException` 的新类：

```js
package com.packtpub.microservices.ch06.attachment.exceptions;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(code = HttpStatus.NOT_FOUND, reason = "No attachment(s) found")
public class AttachmentNotFoundException extends RuntimeException {}
```

1.  我们将在控制器中将所有内容组合在一起。在这个基本示例中，定义了两个方法；一个用于列出特定消息的附件，另一个用于创建新的附件。附件将上传到 Amazon S3 存储桶，其名称在配置值中指定。创建一个名为`com.packtpub.microservices.ch06.attachment.controllers`的新包和一个名为`AttachmentController`的新类：

```js
package com.packtpub.microservices.ch06.attachment.controllers;

import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.model.CannedAccessControlList;
import com.amazonaws.services.s3.model.ObjectMetadata;
import com.amazonaws.services.s3.model.PutObjectRequest;
import com.packtpub.microservices.ch06.attachment.data.AttachmentRepository;
import com.packtpub.microservices.ch06.attachment.exceptions.AttachmentNotFoundException;
import com.packtpub.microservices.ch06.attachment.models.Attachment;
import com.packtpub.microservices.ch06.attachment.models.AttachmentRequest;
import org.apache.commons.codec.binary.Base64;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;

import java.io.ByteArrayInputStream;
import java.io.InputStream;
import java.util.List;

@RestController
public class AttachmentController {

    @Autowired
    private AttachmentRepository attachmentRepository;

    @Autowired
    private AmazonS3 s3Client;

    @Value("${s3.bucket-name}")
    private String bucketName;

    @RequestMapping(path = "/message/{message_id}/attachments", method = RequestMethod.GET, produces = "application/json")
    public List<Attachment> getAttachments(@PathVariable("message_id") String messageId) {
        List<Attachment> attachments = attachmentRepository.findByMessageId(messageId);
        if (attachments.isEmpty()) {
            throw new AttachmentNotFoundException();
        }
        return attachments;
    }

    @RequestMapping(path = "/message/{message_id}/attachments", method = RequestMethod.POST, produces = "application/json")
    public Attachment create(@PathVariable("message_id") String messageId, @RequestBody AttachmentRequest request) {

        byte[] byteArray = Base64.decodeBase64(request.getData());

        ObjectMetadata metadata = new ObjectMetadata();
        metadata.setContentLength(byteArray.length);
        metadata.setContentType("image/jpeg");
        metadata.setCacheControl("public, max-age=31536000");
        InputStream stream = new ByteArrayInputStream(byteArray);

        String fullyResolved = String.format("%s/%s", messageId, request.getFileName());

        s3Client.putObject(
            new PutObjectRequest(bucketName, fullyResolved, stream, metadata)
                .withCannedAcl(CannedAccessControlList.PublicRead));

        String url = String.format("https://%s.s3.amazonaws.com/%s", bucketName, fullyResolved);

        Attachment attachment = new Attachment(messageId, url, request.getFileName(), 1);
        attachmentRepository.save(attachment);
        return attachment;
    }
}
```

1.  为了使这一切都能正常工作，我们必须创建一个属性文件。Java 属性文件支持从环境变量获取值的语法，如下面的代码所示。在`src/main/resources`目录中创建一个名为`application.yml`的新文件：

```js
spring:
  jpa.hibernate.ddl-auto: create
  datasource.url: ${DATABASE_URL}
  datasource.username: ${DATABASE_USERNAME}
  datasource.password: ${DATABASE_PASSWORD}

s3:
  bucket-name: ${BUCKET_NAME}
```

这个示例已经足够好。AWS SDK 中的`EnvironmentVariableCredentialsProvider`期望`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`被设置，我们指定许多非敏感配置值应该从环境变量中读取。这显然比硬编码配置值要好，但我们仍然将秘密暴露给了在同一个容器或进程空间中运行的所有进程。环境变量也必须在某个地方设置（通过配置管理系统或指定在 Dockerfile 中），所以我们还没有解决存储敏感秘密的问题。接下来，我们将修改我们的新服务以从 Vault 读取 S3 凭证。

在这个菜谱中，我们将以开发模式运行 Vault。为生产使用安装 Vault 是一个大主题，无法在一个菜谱中适当覆盖。有关 Vault 的生产使用，请参阅[`www.vaultproject.io/intro/index.html`](https://www.vaultproject.io/intro/index.html)提供的优秀文档。

1.  在你的本地开发机器上安装`vault`。有关任何平台的说明，请参阅[`www.vaultproject.io`](http://www.vaultproject.io)。如果你运行的是 macOS X 并且使用**HomeBrew**，你可以使用单个命令安装 Vault：

```js
$ brew install vault
```

1.  以开发模式运行`vault server`，提供一个易于记住的根令牌：

```js
$ vault server --dev --dev-root-token-id="00000000-0000-0000-0000-000000000000"
```

1.  启用一个新的`kv`秘密引擎实例，其路径特定于此服务：

```js
$ vault secrets enable -path=secret/attachment-service
```

1.  将 AWS 访问密钥和密钥对写入`vault`作为秘密。用你的实际 AWS 访问密钥 ID 和 AWS 密钥访问密钥替换占位符：

```js
$ vault write secret/attachment-service attachment.awsAccessKeyId=<access-key> attachment.awsSecretAccessKey=<access-secret>
```

1.  为了使我们的服务能够从 Vault 读取这些值，我们将使用一个简化 Spring Boot 应用程序 Vault 集成的库。修改我们的项目`build.gradle`文件并添加以下依赖项：

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
apply plugin: 'io.spring.dependency-management'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web'
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-jpa', version: '1.5.9.RELEASE'
 compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-vault-config', version: '1.1.1.RELEASE'
    compile group: 'mysql', name: 'mysql-connector-java'
    compile group: 'com.amazonaws', name: 'aws-java-sdk-s3', version: '1.11.375'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

1.  我们的应用程序需要一个配置类来存储从 Vault 读取的值。创建一个名为`com.packtpub.microservices.ch06.attachment.config`的新包和一个名为`Configuration`的新类：

```js
package com.packtpub.microservices.ch06.attachment.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("attachment")
public class Configuration {

    private String awsAccessKeyId;

    private String awsSecretAccessKey;

    public String getAwsAccessKeyId() {
        return awsAccessKeyId;
    }

    public void setAwsAccessKeyId(String awsAccessKeyId) {
        this.awsAccessKeyId = awsAccessKeyId;
    }

    public String getAwsSecretAccessKey() {
        return awsSecretAccessKey;
    }

    public void setAwsSecretAccessKey(String awsSecretAccessKey) {
        this.awsSecretAccessKey = awsSecretAccessKey;
    }
}
```

1.  修改`Application`类以创建我们刚刚创建的类的实例。然后在创建 S3 客户端时使用该实例，这样我们就可以使用从 Vault 获取的凭证而不是环境变量：

```js
package com.packtpub.microservices.ch06.attachment;

import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.regions.Regions;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;

import com.packtpub.microservices.ch06.attachment.config.Configuration;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
@EnableConfigurationProperties(Configuration.class)
public class Application {

 private final Configuration config;

 public Application(Configuration config) {
 this.config = config;
    }

    @Bean
    public AmazonS3 getS3Client() {
        AmazonS3ClientBuilder client = AmazonS3ClientBuilder.standard();
        AWSCredentials credentials = new BasicAWSCredentials(config.getAwsAccessKeyId(), config.getAwsSecretAccessKey());
        return client.withCredentials(
 new AWSStaticCredentialsProvider(credentials)).withRegion(Regions.US_WEST_2).build();
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

就这样！附件服务现在已配置为从 Vault 读取 AWS 凭证。

# 安全日志

与跟踪和指标一起，日志是可观察系统的一个基本组成部分（我们将在第七章*监控和可观察性*中更广泛地讨论可观察性）。日志是特定系统中事件的有序、带时间戳的序列。

在微服务架构中，拥有多个服务带来的复杂性增加，使得良好的日志记录变得至关重要。良好的日志的具体标准是主观的，但一般来说，良好的日志应该帮助工程师拼凑出可能导致特定错误状态或错误的可能事件。日志通常按级别组织，这是一个可配置的切换，允许开发者指示服务对发送到日志的信息进行更多或更少的详细说明。

虽然对于观察生产环境中系统的行为至关重要，但日志也可能带来隐私和安全风险。从系统发送到日志的大量信息可能会给潜在的攻击者提供有关您系统用户的信息，或者可以用来攻击系统其他部分的敏感信息，如令牌或密钥。微服务架构将可能的攻击面分散开来，这使得精心规划的日志记录策略变得更加重要，以确定服务应该如何记录信息。

# 基础设施即代码

微服务架构通常需要更频繁地提供计算资源。系统中的节点越多，攻击者可以扫描的可能漏洞的攻击面就越大。最容易让系统变得脆弱的方法之一是失去对库存的跟踪，并让多个异构配置保持活跃。在像 Puppet 或 Ansible 这样的配置管理系统流行之前，通常有一套自定义的 shell 脚本，这些脚本会在系统中*引导*新的服务器。这已经足够好了，但随着系统需求的增长和 shell 脚本的修改，将系统的旧部分更新到不断变化的标准变得难以管理。这种类型的配置漂移通常会使得系统的遗留部分容易受到攻击。配置管理系统通过允许团队使用代码（通常使用声明性语法）来描述系统中的节点应该如何配置，从而解决了许多这些问题。配置管理系统通常不处理实际计算资源的提供，例如计算节点、数据存储或网络存储。

基础设施即代码是通过机器可读的代码文件来管理基础设施的配置和运维过程，而不是手动操作。使用代码来描述基础设施可以有效地进行版本控制、代码审查和系统变更的回滚。能够自动化创建数据库节点或将计算节点添加到集群的过程，可以让开发者从关注应用程序中解放出来，相对有信心地确保他们不会在野外留下旧配置。与不可变基础设施相结合，基础设施即代码为系统免受易受攻击、被遗忘组件的侵害提供了额外的安全网。

在本食谱中，我们将演示如何使用由 HashiCorp 创建的开源工具*Terraform*来配置一组 AWS 资源，包括 EC2 实例和 Redis ElastiCache。我们将确保使用 Terraform 配置的资源在网络访问、备份和其他安全考虑方面共享配置。

# 如何做到这一点...

1.  在使用 Terraform 之前，您必须安装它。安装说明可在[项目网站](https://www.terraform.io/intro/getting-started/install.html)上找到，但如果您正在运行 macOS X 并使用 HomeBrew（[`brew.sh/`](https://brew.sh/)），则可以发出以下命令：

```js
$ brew install terraform
```

1.  创建一个名为`example.tf`的新文件。该文件将包含我们的 EC2 实例和 ElastiCache 实例的配置。我们将使用默认的**亚马逊机器镜像**（**AMI**），并启用每天保留五天的快照：

```js
 provider "aws" {
     access_key = "ACCESS_KEY"
     secret_key = "SECRET_KEY"
     region = "us-east-1"
 }

 resource "aws_instance" "example" {
   ami           = "ami-b374d5a5"
   instance_type = "t2.micro"
 }

 resource "aws_elasticache_cluster" "example" {
   cluster_id           = "cluster-example"
   engine               = "redis"
   node_type            = "cache.m3.medium"
   num_cache_nodes      = 1
   parameter_group_name = "default.redis3.2"
   port                 = 6379
   snapshot_window      = "05:00-09:00"
   snapshot_retention_limit = 5
 }
```

将`ACCESS_KEY`和`SECRET_KEY`替换为有效的 AWS 访问密钥对。

1.  初始化 Terraform。这将安装前面文件中引用的 AWS 提供程序：

```js
$ terraform init
```

1.  Terraform 通过呈现执行计划并询问您是否希望通过应用该计划来继续操作。运行以下命令并在提示时输入`yes`：

```js
$ terraform apply

aws_instance.example: Refreshing state... (ID: i-09b5cf5ed923d60f4)
```

已生成执行计划，如下面的代码所示。资源操作用以下符号表示：`+ create`。

Terraform 将执行以下操作：

```js
+ aws_elasticache_cluster.example
 id: <computed>
 apply_immediately: <computed>
 availability_zone: <computed>
 az_mode: <computed>
 cache_nodes.#: <computed>
 cluster_address: <computed>
 cluster_id: "cluster-example"
 configuration_endpoint: <computed>
 engine: "redis"
 engine_version: <computed>
 maintenance_window: <computed>
 node_type: "cache.m3.medium"
 num_cache_nodes: "1"
 parameter_group_name: <computed>
 port: "6379"
 replication_group_id: <computed>
 security_group_ids.#: <computed>
 security_group_names.#: <computed>
 snapshot_retention_limit: "5"
 snapshot_window: "05:00-09:00"
 subnet_group_name: <computed>

Plan: 1 to add, 0 to change, 0 to destroy.
Do you want to perform these actions?
```

Terraform 将执行前面描述的操作。只有`yes`会被接受以批准：

```js
Enter a value: yes

...
```

1.  登录您的 AWS 管理控制台，您将看到已创建一个新的 Redis 集群和 EC2 实例。Terraform 还可以帮助您清理。为了销毁这两个资源，运行销毁命令并在提示时输入`yes`：

```js
$ terraform destroy
```

Terraform 是一个功能强大的工具。在本食谱中，我们用它创建了一个 EC2 实例和一个 ElastiCache 集群实例。您可以用 Terraform 做更多的事情——基础设施即代码的主题可以填满一本自己的食谱。幸运的是，HashiCorp 提供的文档（[`www.terraform.io/docs/index.html`](https://www.terraform.io/docs/index.html)）非常出色，我建议您阅读它们。

使用基础设施即代码解决方案将使资源配置和管理过程变得更加安全，限制因配置过时而丢失旧有基础设施的可能性。
