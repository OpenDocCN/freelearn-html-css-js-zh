# 第十章：让它上线

在本章中，我们将涵盖：

+   部署到服务器环境

+   自动崩溃恢复

+   持续部署

+   使用平台即服务提供商进行托管

# 介绍

Node 是构建和提供在线服务的绝佳平台选择。无论是简单的、精简的网站，还是高度灵活的 Web 应用程序，或者超越 HTTP 的服务，我们都必须在某个时候部署我们的创作。

本章重点介绍了将我们的 Node 应用程序上线所需的步骤。

# 部署到服务器环境

虚拟专用服务器（VPS）、专用服务器或基础设施即服务（例如，亚马逊 EC2 或 Rackspace 等）以及拥有我们自己的服务器机器都有一个共同点：对服务器环境的完全控制。

然而，伴随着巨大的权力而来的是巨大的责任，我们需要意识到一些挑战。本配方将演示如何克服这些挑战，安全地在端口`80`上初始化一个 Node Web 应用程序。

## 准备就绪

当然，我们需要一个远程服务器环境（或我们自己的设置）。研究找到最适合我们需求的最佳套餐非常重要。

专用服务器可能很昂贵。硬件与软件的比例是一比一，我们实际上是在租用一台机器。

VPS 可能更便宜，因为它们共享单台机器（或集群）的资源，因此我们只租用托管操作系统实例所需的资源。然而，如果我们开始使用超出分配的资源，我们可能会受到处罚（停机时间，额外费用），因为过度使用可能会影响其他 VPS 用户。

IaaS 可能相对便宜，特别是在涉及扩展时（当我们需要更多资源时），尽管 IaaS 往往包含按使用量计费的元素，这意味着成本不是固定的，可能需要额外的监控。

我们的配方假设使用运行`sshd`（SSH 服务）的 Unix/Linux 服务器。此外，我们应该有一个指向我们服务器的域名。在这个配方中，我们将假设域名为`nodecookbook.com`。最后，我们必须在远程服务器上安装 Node。如果出现困难，我们可以使用[`www.github.com/joyent/node/wiki/Installation`](https://www.github.com/joyent/node/wiki/Installation)上提供的说明，或者通过包管理器安装，我们可以使用[`www.github.com/joyent/node/wiki/Installing-Node.js-via-package-manager`](https://www.github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)上的说明。

我们将从第六章*使用 Express 加速开发*的倒数第二个配方中部署`login`应用程序，所以我们需要这个。

## 如何做...

为了准备我们的应用程序传输到远程服务器，我们将删除`node_modules`文件夹（我们可以在服务器上重建它）：

```js
rm -fr login/node_modules 

```

然后我们通过执行以下命令压缩`login`目录：

```js
npm pack login 

```

这将生成一个压缩的存档，名称与`package.json`文件中给出的应用程序名称和版本相同，对于未经修改的 Express 生成的`package.json`文件，将生成文件名`application-name-0.0.1.tgz`。

无论`npm pack`称其为什么，让我们将其重命名为`login.tgz`：

```js
mv application-name-0.0.1.tgz login.tgz #Linux/Mac OS X
rename application-name-0.0.1.tgz login.tgz ::Windows. 

```

接下来，我们将`login.tgz`上传到我们的服务器。例如，我们可以使用 SFTP：

```js
sftp root@nodecookbook.com 

```

一旦通过 SFTP 登录，我们可以发出以下命令：

```js
cd /var/www
put login.tgz 

```

将上传到`/var/www`目录并不是必需的，这只是放置网站的一个自然位置。

这假设我们已经通过 SFTP 从包含`login.tgz`的目录 SFTP 到我们的服务器。 

接下来，我们通过 SSH 登录到服务器：

```js
ssh -l root nodecookbook.com 

```

### 提示

如果我们使用 Windows 桌面，我们可以使用 putty 进行 SFTP 和 SSH 登录到我们的服务器：[`www.chiark.greenend.org.uk/~sgtatham/putty/`](http://www.chiark.greenend.org.uk/~sgtatham/putty/)。

一旦登录到远程服务器，我们就导航到`/var/www`并解压`login.tar.gz`：

```js
tar -xvf login.tar.gz 

```

当`login.tar.gz`解压缩时，它会在服务器上重新创建我们的`login`文件夹。

要重建`node_modules`文件夹，我们进入`login`文件夹并使用`npm`重新生成依赖项。

```js
cd login
npm -d install

```

大多数服务器都有基于 shell 的编辑器，如`nano，vim`或`emacs`。我们可以使用这些编辑器之一来更改`app.js`中的一行（或者通过 SFTP 传输修改后的`app.js`）：

```js
app.listen(80, function () { process.setuid('www-data'); });

```

我们现在正在监听标准 HTTP 端口，这意味着我们可以访问我们的应用程序而无需在其 Web 地址后加上端口号。但是，由于我们将以`root`身份启动应用程序（为了绑定到端口`80`是必要的），我们还将回调传递给`listen`方法，该方法将应用程序的访问权限从`root`更改为`www-data`。

在某些情况下，根据文件权限，从我们的应用程序读取或写入文件可能不再起作用。我们可以通过更改所有权来解决这个问题：

```js
chown -R www-data login 

```

最后，我们可以用以下方式启动我们的应用程序：

```js
cd login
nohup node app.js & 

```

我们可以确保我们的应用程序正在作为`www-data`运行：

```js
ps -ef | grep node 

```

## 它是如何工作的...

我们修改了`app.listen`以绑定到端口`80`，并添加了一个回调函数，该函数将用户 ID 从`root`重置为`www-data`。

向`listen`添加回调不仅限于 Express，它在使用普通的`httpServer`实例时也是一样的。

以`root`身份运行 Web 服务器是不好的做法。如果我们的应用程序被攻击者入侵，他们将通过我们应用程序的特权状态获得对我们系统的`root`访问权限。

为了降级我们的应用程序，我们调用`process.setuid`并传入`www-data. process.setuid`。这要么是用户的名称，要么是用户的 UID。通过传递一个名称，我们导致`process.setuid`阻塞事件循环（基本上冻结操作），同时它交叉引用用户字符串到其 UID。这消除了应用程序绑定到端口`80`并作为`root`运行的潜在时间。实质上，将字符串传递给`process.setuid`而不是底层 UID 意味着在应用程序不再是`root`之前什么都不会发生。

我们使用`nohup`调用我们的进程，然后跟上`&`。这意味着我们可以自由结束我们的 SSH 会话，而不会导致我们的应用程序随着会话终止而终止。

安德符号将我们的进程转换为后台任务，因此我们可以在其运行时做其他事情（比如`退出`）。`nohup`意味着忽略挂断信号（HUP）。每当 SSH 会话终止时，HUP 被发送到通过 SSH 启动的任何运行进程。基本上，使用`nohup`允许我们的 Web 应用程序在 SSH 会话结束后继续存在。

## 还有更多...

有其他方法可以独立于我们的会话启动我们的应用程序，并绑定到端口`80`而不以`root`身份运行应用程序。此外，我们还可以运行多个应用程序并使用`http-proxy`将它们代理到端口`80`。

### 使用`screen`而不是`nohup`

实现与使用`nohup`独立于我们的 SSH 会话的另一种方法是使用`screen`。我们将使用它如下：

```js
screen -S myAppName 

```

这将给我们一个虚拟终端，我们可以说：

```js
cd login
node app.js 

```

然后我们可以通过按下`Ctrl + A`，然后按`D`来离开虚拟终端。我们将返回到我们最初的终端。虚拟终端将在我们注销 SSH 后继续运行。我们随时可以重新登录 SSH 并说：

```js
screen -r myAppName 

```

在那里我们可以看到任何控制台输出并停止`(Ctrl + C)`和启动应用程序。

### 使用特权端口的 authbind

在这个例子中，我们应该以非 root 用户的身份 SSH 到我们的服务器：

```js
ssh -l dave nodecookbook.com 

```

绑定到端口`80`的另一种方法是使用`authbind`，可以通过我们服务器的软件包管理器安装。例如，如果我们的软件包管理器是`apt-get`，我们可以说：

```js
sudo apt-get install authbind 

```

`authbind`通过抢占端口绑定的操作系统策略并在执行时利用一个名为`LD_PRELOAD`的环境变量来工作。因此，它永远不需要以`root`权限运行。

为了使它为我们工作，我们必须进行一些简单的配置工作，如下所示：

```js
sudo touch /etc/authbind/byport 80
sudo chown dave /etc/authbind/byport 80
sudo chmod 500 /etc/authbind/byport 80 

```

这告诉`authbind`允许用户`dave`将进程绑定到端口`80`。

我们不再需要更改进程 UID，因此我们编辑`app.js`的倒数第二行为：

```js
app.listen(80);

```

我们还应该按以下方式更改`login`文件夹的所有权：

```js
chown -R dave login 

```

现在我们可以在完全不触及`root`访问的情况下启动我们的服务器：

```js
nohup authbind node app.js & 

```

`authbind`可以使我们的应用立即运行，无需任何修改。但是，它目前缺乏 IPv6 支持，因此尚不具备未来的兼容性。

### 从端口 80 托管多个进程

如何使用默认 HTTP 端口提供多个进程？

我们可以使用第三方`http-proxy`模块来实现这一点。

```js
npm install http-proxy 

```

假设我们有两个应用程序，一个（我们的`login`应用程序）托管在`login.nodecookbook.com`，另一个（本书第一个示例中的`server.js`文件）简单地托管在`nodecookbook.com`。这两个域指向同一个 IP。

`server.js`将监听端口`8080`，我们将修改`login/app.js`以再次监听端口`3000`，如下面的代码所示：

```js
app.listen(3000, '127.0.0.1');

```

我们还添加了第二个参数，定义绑定到哪个地址（而不是任何地址）。这可以防止我们的服务器通过端口被访问。

让我们在一个新文件夹中创建一个文件，称之为`proxy.js`，并写入以下内容：

```js
require('http-proxy')
  .createServer({router : {
    'login.nodecookbook.com': 'localhost:3000',
    'nodecookbook.com': 'localhost:8080'
  }}).listen(80, function () {
    process.setuid('www-data');
  });

```

`createServer`传递的对象包含一个路由器属性，该属性本身是一个对象，指示`http-proxy`根据其端口将特定域上的传入流量路由到正确的本地托管进程。

最后，我们绑定到端口`80`，并从`root`降级到`www-data`。

要初始化，我们必须执行：

```js
nohup node login/app.js &
nohup node server.js &
nohup node proxy.js & 

```

由于我们将代理服务器绑定到端口`80`，这些命令必须以`root`身份运行。如果我们正在使用非 root 帐户操作 SSH，我们只需在这三个命令前加上`sudo`。

## 另请参阅

+   *本章讨论的自动崩溃恢复*

+   *本章讨论的持续部署*

+   *本章讨论的作为服务提供商的平台托管*

# 自动崩溃恢复

当我们创建一个站点时，服务器和站点逻辑都绑定在一个进程中。而在其他平台上，服务器代码已经就位。如果我们的站点代码有错误，服务器很不可能崩溃，因此在许多情况下，即使其中一部分出现问题，站点也可以保持活动状态。

对于基于 Node 的网站，一个小错误可能会导致整个进程崩溃，而这个错误可能只会在很长时间内触发一次。

作为一个假设的例子，错误可能与 POST 请求的字符编码有关。当像 Felix Geisendörfer 这样的人完成并提交表单时，突然间我们整个服务器崩溃了，因为它无法处理变音符号。

在这个示例中，我们将使用 Upstart，这是一个可用于 Linux 服务器的事件驱动的 init 服务，它不是基于 Node，但仍然是一个非常方便的助手。

## 准备工作

我们需要在服务器上安装 Upstart。[`upstart.ubuntu.com`](http://upstart.ubuntu.com)包含有关如何下载和安装的说明。如果我们已经在使用 Ubuntu 或 Fedora 远程服务器，则 Upstart 将已经集成。

## 如何做...

让我们创建一个新的服务器，当我们通过 HTTP 访问它时故意崩溃：

```js
var http = require('http');
http.createServer(function (req, res) {
  res.end("Oh oh! Looks like I'm going to crash...");
  throw crashAhoy;
}).listen(8080);

```

在第一页加载后，服务器将崩溃，站点将下线。

让我们将这段代码称为`server.js`，将其放在远程服务器上的`/var/www/crashingserver`下

现在我们创建我们的 Upstart 配置文件，将其保存在服务器上的`/etc/init/crashingserver.conf`。

```js
start on started network-services

respawn
respawn limit 100 5

setuid www-data

exec /usr/bin/node /var/www/crashingserver/server.js >> \ /var/log/crashingserver.log 2>&1 

post-start exec echo "Server was (re)started on $(date)" | mail -s "Crashing Server (re)starting" dave@nodecookbook.com

```

最后，我们初始化我们的服务器如下：

```js
start crashingserver 

```

当我们访问`http://nodecookbook.com:8080`并刷新页面时，我们的网站仍然可以访问。快速查看`/var/log/crashingserver.log`，我们可以发现服务器确实崩溃了。我们还可以检查我们的收件箱以查找服务器重新启动通知。

## 它是如何工作的...

Upstart 服务的名称取自特定的 Upstart 配置文件名。我们使用`start crashingserver`来启动`/etc/init/crashingserver.conf` Upstart 服务。

配置的第一行确保我们的 Web 服务器在远程服务器的操作系统重新启动时自动恢复（例如，由于停电或需要重新启动等）。

`respawn`被声明两次，一次用于打开重生，然后设置一个“重生限制 - 每 5 秒最多 100 次重启”。限制必须根据我们自己的情况进行设置。如果网站流量较低，这个数字可能会调整为在 8 秒内重启 10 次。

我们希望尽可能保持在线，但如果问题持续存在，我们可以将其视为一个警示，表明错误对用户体验或系统资源产生了不利影响。

下一行将我们的服务器初始化为`www-data`用户，并将输出发送到`/var/log/crashingserver.log`。

最后一行在我们的服务器启动或重新启动后立即发送电子邮件。这样我们就可以得知可能需要解决服务器问题。

## 还有更多...

让我们实现另一个 Upstart 脚本，如果服务器崩溃超出其“重生限制”，我们还将看另一种方法来保持我们的服务器在线。

### 检测重生限制违规

如果我们的服务器超出了“重生限制”，很可能存在严重问题，应尽快解决。我们需要立即了解情况。为了在 Upstart 中实现这一点，我们可以创建另一个 Upstart 配置文件，监视`crashingserver`守护程序，如果“重生限制”被违反，则发送电子邮件。

```js
task

start on stopped crashingserver PROCESS=respawn

script
  if [ "$JOB" != ''  ]
    then echo "Server "$JOB" has crashed on $(date)" | mail -s \
    $JOB" site down!!" dave@nodecookbook.com
  fi
end script

```

让我们把它保存到`/etc/init/sitedownmon.conf`。

然后我们做：

```js
start crashingserver
start sitedownmon 

```

我们将这个 Upstart 进程定义为一个任务（它只有一件事要做，之后就退出了）。我们不希望在我们的服务器崩溃后它继续存在。

当`crashingserver`守护程序在重生期间停止时执行该任务（例如，当“重生限制”被打破时）。

我们的脚本段（指令）包含一个小的 bash 脚本，用于检查`JOB`环境变量的存在（在我们的情况下，它将设置为`crashingserver`），然后相应地发送电子邮件。如果我们不检查它的存在，当它首次启动并发送一个带有空`JOB`变量的电子邮件时，`sitedownmon`似乎会触发错误的警报。

稍后我们可以通过在每个服务器的`sitedownmon.conf`中添加一行来扩展此脚本：

```js
start on stopped anotherserver PROCESS=respawn

```

### 使用 forever 保持在线

有一个更简单的基于 Node 的替代方案叫做`forever:`

```js
npm -g install forever 

```

如果我们只是用以下方式启动我们的服务器：

```js
forever server.js 

```

然后访问我们的网站，一些终端输出将包含以下内容：

```js
warn: Forever detected script exited with code: 1
warn: Forever restarting script for 1 time

```

但我们仍然可以访问我们的网站（尽管它已经崩溃并重新启动）。

要在远程服务器上部署我们的网站，我们通过 SSH 登录到服务器，安装`forever`并说：

```js
forever start server.js 

```

虽然这种技术确实较少复杂，但也较不稳健。Upstart 提供了核心内核功能，因此是系统关键性的。如果 Upstart 失败，内核就会发生恐慌，整个服务器就会重新启动。

然而，在 Nodejitsu 的 PaaS 堆栈上广泛使用`forever`，其吸引人的简单性可能适用于不太关键的生产环境。

## 另请参阅

+   本章讨论了*部署到服务器环境*

+   本章讨论了使用平台即服务提供商进行托管

+   本章讨论了*持续部署*

# 持续部署

我们的流程越简化，我们就能更加高效。持续部署是指将小的持续改进提交到生产服务器，以节省时间、高效地进行。

持续部署对团队协作项目尤为重要。与其在代码的不同分支上工作并花费额外的时间、金钱和精力进行集成，不如让每个人都在同一代码库上工作，这样集成就会更加顺畅。

在这个示例中，我们将使用 Git 作为版本控制工具创建部署流程。虽然这可能不是 Node，但它肯定可以提高编码、部署和管理 Node 项目的生产力。

### 注意

如果我们对 Git 有点陌生，我们可以从 Github 的帮助文档中获得见解，[`help.github.com`](http://help.github.com)。

## 准备就绪

我们需要在服务器和桌面系统上都安装 Git，不同系统的说明可以在这里找到[`book.git-scm.com/2_installing_git.html`](http://book.git-scm.com/2_installing_git.html)。如果我们使用带有`apt-get`软件包管理器的 Linux，我们可以执行：

```js
sudo apt-get install git git-core 

```

如果我们是第一次安装 Git，我们将不得不按照以下方式设置个人信息配置设置：

```js
git config --global user.name "Dave Clements"
git config --global user.email "dave@nodecookbook.com" 

```

我们将使用我们的`login`应用程序，在第一个教程中我们将其部署到服务器上。因此，让我们 SSH 到服务器并进入`/var/www/login`目录。

```js
ssh -l root nodecookbook.com -t "cd /var/www/login; bash" 

```

由于我们将不会以 root 身份运行我们的应用程序，因此我们将简化事情并将`login/app.js`中的监听端口更改为`8000`：

```js
app.listen(8000);

```

## 如何做...

一旦我们登录到服务器并在`login`文件夹中安装了 Git（请参阅*准备工作*），我们说：

```js
git init
git add *
git commit -m "initial commit" 

```

接下来，我们创建一个裸存储库（它记录了所有更改，但没有实际的工作文件），我们将向其推送更改。这有助于保持一致。

我们将称这个裸存储库为`repo`，因为这是我们将推送更改的存储库，并且我们将在`login`文件夹中创建它：

```js
mkdir repo
echo repo > .gitignore
cd repo
git --bare init 

```

接下来，我们将裸`repo`连接到`login`应用程序存储库，并将所有提交推送到`repo`。

```js
cd ..
git remote add repo ./repo
git push repo master 

```

现在我们将编写一个 Git 挂钩，指示`login`存储库从裸`repo`存储库中拉取任何更改，然后在通过远程 Git 推送更新`repo`时重新启动我们的`login`应用程序。

```js
cd repo/hooks
touch post-update
chmod +x post-update
nano post-update 

```

在`nano`中打开文件后，我们编写以下代码：

```js
#!/bin/sh

cd /root/login
env -i git pull repo master

exec forever restart /root/login/app.js

```

使用*Ctrl* + *O*保存我们的挂钩，然后使用*Ctrl* + *X*退出。

如果我们对`login`存储库进行 Git 提交，这两个存储库可能会不同步。为了解决这个问题，我们为`login`存储库创建另一个挂钩：

```js
#!/bin/sh
git push repo

```

我们将其存储在`login/.git/hooks/post-commit`中，并确保使用`chmod +x post-commit`使其可执行。

我们将通过 SSH 协议远程向`repo`进行提交。理想情况下，我们希望为 Git 交互创建一个系统用户。

```js
useradd git
passwd git   #set a password

mkdir /home/git
chown git /home/git 

```

我们还为`git`用户创建了主目录，以便`forever`可以轻松存储日志和 PID 文件。我们需要将`git`设置为`login`应用程序的所有者，以便我们可以通过 SSH 使用 Git 来管理它：

```js
cd /var/www
chown -R git login 

```

最后（对于服务器端设置），我们以`git`用户身份登录并使用`forever`启动我们的应用程序。

```js
su git
forever start /var/www/login/app.js 

```

假设我们的服务器托管在`nodecookbook.com`，我们现在可以在`http://nodecookbook.com:8000`访问`login`应用程序。

回到桌面，我们克隆`repo`存储库：

```js
git clone ssh://git@nodecookbook.com/var/www/login/repo 

```

这将给我们一个`repo`目录，其中包含与我们原始的`login`文件夹完全匹配的所有生成的文件。然后我们可以进入`repo`文件夹并更改我们的代码（比如，在`app.js`中更改端口）。

```js
app.listen(9000);

```

然后我们提交更改并推送到我们的服务器。

```js
git commit -a -m "changed port"
git push 

```

在服务器端，我们的应用程序应该已自动重新启动，因此我们的应用程序现在是从`http://nodecookbook.com:9000`而不是`http://nodecookbook.com:8000`托管的。

## 它是如何工作的...

我们创建了两个 Git 存储库。第一个是`login`应用程序本身。当我们运行`git init`时，`.git`目录将添加到`login`文件夹中。`git add *`添加文件夹中的所有文件，`commit -m "initial commit"`将我们的添加放入 Git 的版本控制系统中。因此，现在我们的整个代码库都被 Git 识别。

第二个是`repo`，它是使用`--bare`标志创建的。这是一种骨架存储库，提供了所有预期的 Git 功能，但缺少实际文件（它没有工作树）。

虽然使用两个存储库可能看起来过于复杂，但实际上大大简化了事情。由于 Git 不允许将推送到当前签出的分支，因此我们必须创建一个单独的虚拟分支，以便我们可以从主分支签出并进入虚拟分支。这会导致 Git 挂钩和重新启动我们的应用程序出现问题。挂钩尝试启动错误的分支的应用程序。分支也很快会不同步，而挂钩只会火上浇油。

由于`repo`位于`login`目录中，我们创建一个`.gitignore`文件，告诉 Git 忽略这个子目录。尽管`login`和`repo`在同一台服务器上，我们将`repo`添加为`remote`存储库。这在存储库之间增加了一些必要的距离，并允许我们稍后使用我们的第一个 Git 钩子使`login`从`repo`拉取更改。从`repo`到`login`的 Git 推送不会导致`login`更新其工作目录，而从`repo`到`login`的拉取确实会启动合并。

在我们的`remote add`之后，我们从主分支（login）向`repo`执行初始推送，现在它们在同一张乐谱上演奏。

然后我们创建了我们的钩子。

Git 钩子是可执行文件，驻留在存储库的`hook`文件夹中。有各种可用的钩子（已经在文件夹中，但后缀为`.sample`）。我们使用了两个：`post-update`和`post-commit`。一个在更新后执行（例如，一旦更改已被拉取并集成到`repo`中），另一个在提交后执行。

第一个钩子`login/repo/hooks/post-update`基本上提供了我们的持续部署功能。它使用`cd`将其工作目录从`repo`更改为`login`，并命令`git pull`。`git pull`命令前缀为`env -i`。这可以防止某些 Git 功能出现问题，否则会执行 Git 命令代表`repo`，无论我们将我们的钩子脚本发送到什么目录。Git 利用`$GIT_DIR`环境变量将我们锁定到调用钩子的存储库。`env -i`通过告诉`git pull`忽略（`-i`）所有环境变量来处理这个问题。

更新工作目录后，我们的钩子继续调用`forever restart`，从而使我们的应用程序重新初始化并应用提交的更改。

我们的第二个钩子只是一个填充物，以确保在直接提交到`login`存储库时代码库的一致性。直接向`login`目录提交不会更新工作树，也不会导致我们的应用程序重新启动，但`login`和`repo`之间的代码至少会保持同步。

为了限制损害（如果我们曾经受到攻击），我们为处理 SSH 上的 Git 更新创建了一个特定的账户，为其提供一个主目录，接管`login`应用程序并执行我们应用程序的主要初始化。

一旦服务器配置完成，一切都很顺利。在将`repo`存储库克隆到我们的本地开发环境后，我们只需进行更改，添加和提交，然后推送到服务器。

服务器接收我们的推送请求，更新`repo`，启动`post-update`钩子，使`login`从`repo`拉取更改，之后`post-update`钩子使用`forever`重新启动`app.js`，因此我们有了一个持续部署工作流程。

我们可以从任意位置克隆任意数量的克隆，因此这种方法非常适合于地理位置独立的团队协作项目，无论规模大小。

## 还有更多...

我们可以通过在 post-update 钩子中使用`npm install`来避免上传模块。此外，Git 钩子不一定要用 shell 脚本编写，我们可以用 Node 来编写它们！

### 构建模块依赖关系的更新

一些 Node 模块是纯 JavaScript 编写的，另一些具有 C 或 C++绑定。具有 C 或 C++绑定的模块必须从源代码构建-这是一个特定于系统的任务。除非我们的实时服务器环境与我们的开发环境完全相同，否则我们不应该简单地将为一个系统构建的代码推送到另一个系统上。

另外，为了节省传输带宽并实现更快的同步，我们可以让我们的 Git 钩子安装所有模块（本地绑定和 JavaScript），并让 Git 完全忽略`node_modules`文件夹。

因此，在我们的本地存储库中，让我们做以下事情：

```js
echo node_modules >> .gitignore 

```

然后我们将我们裸远程存储库（login/repo/hooks）中的`post-update`钩子更改为：

```js
#!/bin/sh

cd /root/login

env -i git pull repo master && npm rebuild && npm install

exec forever restart /root/login/app.js

```

我们已经在`git pull`行中添加了`&& npm rebuild && npm install`（使用`&&`确保它们受益于`env -i`命令）。

现在，如果我们向`package.json`添加了一个模块，并执行了`git commit -a`，然后执行`git push`，我们的本地`repo`将`package.json`推送到远程`repo`。这将触发`post-update`挂钩将更改拉入主`login`存储库，并随后执行`npm rebuild`（重新构建任何 C / C++依赖项）和`npm install`（安装任何新模块）。

### 编写一个用于集成测试的 Node Git 挂钩

持续部署是持续集成的延伸，通常期望对任何代码更改运行彻底的测试套件以进行质量保证。

我们的`login`应用（作为一个基本的演示站点）没有测试套件（有关测试套件的信息，请参见第九章中，*编写自己的 Node 模块*），但我们仍然可以编写一个挂钩，以便在将来为`login`添加任何测试时执行。

此外，我们可以用 Node 编写它，这样做的额外好处是可以跨平台运行（例如在 Windows 上）。

```js
#!/usr/bin/env node

var npm = require("npm");

npm.load(function (err) {
    if (err) { throw err; }

    npm.commands.test(function (err) {
        if (err) { process.exit(1); }       
    });

});

```

我们将把这段代码放在服务器上的`login/repo/hooks/pre-commit`中，并使其可执行（`chmod +x pre-commit`）。

第一行将`node`设置为脚本解释器指令（就像`#!/bin/sh`为 shell 脚本设置`sh` shell 一样）。现在我们进入了 Node 的领域。

我们使用`npm`的可编程性，加载我们应用的`package.json`文件，然后运行测试脚本（如果有指定的话）。

然后，我们将以下内容添加到我们的`package.json`文件中：

```js
{
    "name": "application-name"
  , "version": "0.0.1"
  , "private": true
  , "dependencies": {
      "express": "2.5.5"
    , "jade": ">= 0.0.1"
  },
   "scripts": {
    "test": "node test"
  },
  "devDependencies": {"npm": "1.1.18"}
}

```

然后执行以下操作：

```js
npm -d install 

```

现在，每当我们推送到`repo`时，只有通过测试的更改才会被提交。只要我们有一个良好编写的测试套件，这是保持良好代码的好方法。

### 提示

对于我们的`scripts.test`属性，我们使用了`node test`（就像在第九章中，*编写自己的 Node 模块*中一样）。然而，我们还可以使用更高级的测试框架，比如 Mocha [`visionmedia.github.com/mocha/`](http://visionmedia.github.com/mocha/)。

### 注意

这个 Node Git 挂钩是根据 Domenic Denicola 的一个 gist（经过许可）进行调整的，可以在[`gist.github.com/2238951`](https://gist.github.com/2238951)找到。

## 另请参阅

+   本章讨论的*部署到服务器环境*

+   本章讨论的*自动崩溃恢复*

+   创建一个测试驱动的模块 API，在第九章中讨论，编写自己的 Node 模块

+   本章讨论的*使用平台即服务提供商进行托管*

# 使用平台即服务提供商进行托管

Node 的**平台即服务提供商（PaaS）**包含了前三章讨论的所有概念，并将部署简化为一组非常基本但强大的命令。在部署方面，PaaS 可以让我们的生活变得非常简单。只需一个简单的命令，我们的应用就可以部署，另一个命令可以无缝更新和重新初始化。

在这个例子中，我们将学习如何部署到 Nodejitsu，这是领先的 Node 托管平台提供商之一。

## 做好准备

首先，我们将安装`jitsu`，Nodejitsu 的部署和应用管理命令行应用程序。

```js
sudo npm -g install jitsu 

```

在继续之前，我们必须按照以下步骤注册一个帐户：

```js
jitsu signup 

```

该应用程序将引导我们完成简单的注册过程，并为我们创建一个帐户，我们必须通过电子邮件确认。

### 提示

Nodejitsu 并不是唯一的 Node PaaS，还有其他类似的平台，如 no.de、Nodester 和 Cloud Foundry，它们遵循类似的流程。

一旦我们收到了我们的电子邮件，我们就可以使用提供的凭证，例如：

```js
jitsu users confirm daveclements _sCjXz46in-6IBpl 

```

与第一个示例一样，我们将使用第六章中的*初始化和使用会话*配方中的`login`应用程序，使用`login`应用程序。

## 如何做...

首先，我们进入`login`文件夹并对`package.json`进行一些修改：

```js
{
  "name": "ncb-login",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "express": "2.5.5",
    "jade": ">= 0.0.1"
  },
  "subdomain": "login",
  "scripts": {
    "start": "app.js"
  },
  "engines": {
    "node": "0.6.x"
  }
}

```

现在我们部署！

```js
Jitsu deploy

```

如果我们在`http://login.nodejitsu.com`或者`http://login.jit.su`导航到我们指定的子域，我们将看到我们的`login`应用程序（如果子域不可用，`jitsu`将建议替代方案）。

## 它是如何工作的...

我们对`package.json`进行了一些修改。我们的应用程序名称是唯一必须直接编辑`package.json`进行的更改。其他添加可能已经由`jitsu`可执行文件代表我们完成。设置应用程序的名称很重要，因为在`jitsu`中，应用程序是通过其名称进行管理的。

如果我们没有将`subdomain, scripts`和`engines`属性附加到`package.json`中，当我们运行`jitsu deploy`并由`jitsu`代表我们重新生成`package.json`时，`jitsu`将要求我们提供详细信息。

`subdomain`指定了`nodejistu.com`的标签前缀，我们从中托管我们的应用程序（例如，`login.nodejitsu.com`）。`scripts`，带有`start`子属性，通知 Nodejitsu 我们的启动脚本，启动应用程序的文件。`engines`定义了我们的应用程序设计的 Node 的哪些版本。

## 还有更多...

让我们看看如何通过自定义域名访问我们的 Nodejitsu 应用，并通过`jitsu`可执行文件为其提供数据库后端。

### 为 Nodejitsu 应用分配自定义域名

为了为我们的应用程序准备通过自定义域名提供服务，我们对`package.json`进行了修改，如下所示：

```js
//prior package.json data
 "subdomain": "login",
  "domains": "login.nodecookbook.com",
  "scripts": {
    "start": "app.js"
  },
//rest of package.json data

```

然后我们使用`jitsu`推送我们的更改，如下所示：

```js
jitsu apps update ncb-login 

```

现在应用程序已准备好通过[`login.nodecookbook.com`](http://login.nodecookbook.com)接收流量，但在流量到达之前，我们必须将我们的域的 A 记录与 Nodejitsu 的 A 记录匹配。

我们可以使用`dig`（或类似的命令行应用程序）获取当前的 Nodejitsu A 记录列表：

```js
dig nodejitsu.com 

```

更改 A 记录的过程取决于我们的域名提供商。通常可以在提供商的控制面板/管理区域的 DNS 区域找到它。

### 使用 jitsu 为数据库提供服务

在第六章的最后一个配方中，*使用 Express 加速开发*，我们构建了一个使用 MongoDB 支持的 Express 应用程序。现在我们将使用 Nodejitsu 将`profiler`应用程序上线，并利用`jitsu`的数据库提供功能。

所以让我们为`profiler`数据库提供一个 Mongo 数据库，如下所示：

```js
jitsu databases create mongo profiler 

```

`jitsu`将通过第三方数据库 PaaS 提供商（在 Mongo 的情况下，PaaS 提供商是 MongoHQ）为我们提供数据库。输出的倒数第二行为我们提供了新数据库的 MongoDB URI，看起来像以下代码：

```js
info: Connection url: mongodb://nodejitsu:14dce01bda24e5fe53bbdaa8f2f6547b@flame.mongohq.com:10019/nodejitsudb169742247544

```

因此，我们将`profiler/tools/prepopulate.js`的第二行更新为：

```js
client = mongo.db('mongodb://nodejitsu:14dce01bda24e5fe53bbdaa8f2f6547b@flame.mongohq.com:10019/nodejitsudb169742247544'),

```

然后我们从`profiler/tools`文件夹运行它：

```js
node prepulate.js 

```

这将填充我们的远程数据库与配置文件和登录数据。

我们在另外两个地方`profiler/profiles.js`和`profiler/login/login.js`中更新了我们的数据库 URI，在这两个地方，第二行被修改为：

```js
db = mongo.db('mongodb://nodejitsu:14dce01bda24e5fe53bbdaa8f2f6547b@flame.mongohq.com:10019/nodejitsudb169742247544'),

```

最后，我们输入以下内容：

```js
jitsu deploy 

```

`jitsu`将要求我们设置某些设置（`子域，scripts.start`和`engines`），我们可以只需按下*Enter*并使用默认设置（除非`profiler.nodejitsu.com`已被占用，这种情况下我们应该选择不同的 URL）。

然后`jitsu`将部署我们的应用程序，我们应该能够在`profiler.nodejitsu.com`上访问它。

## 另请参阅

+   *在本章讨论的*部署到服务器环境

+   *在本章讨论的*自动崩溃恢复

+   *在本章讨论的*持续部署
