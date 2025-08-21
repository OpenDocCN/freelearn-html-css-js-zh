# 第八章：服务器端调优

在本章中，我们将涵盖：

+   防止移动转码

+   添加移动 MIME 类型

+   使缓存清单正确显示

+   设置远期过期标头

+   Gzip 压缩

+   实体标签移除

# 介绍

服务器端性能直接影响页面加载速度。适当的服务器配置可以极大地提高客户端加载速度。

在本章中，我们将介绍一些用于使移动网站和应用程序性能更好更快的服务器端配置。一些概念是移动中心的；一些也适用于桌面网络。

有许多服务器最佳实践指南，但有些可能不够全面。在本章中，我们将结合最佳实践，看看如何最大化网站的性能。

# 防止移动转码

目标浏览器：跨浏览器

许多移动运营商使用代理或适配引擎来更改您要提供的网页内容。在许多移动设备上，内置或安装的浏览器使用移动转码器来重新格式化和压缩页面内容。这被称为**移动转码**。如果您不希望内容被更改，必须添加 HTTP 头部以防止移动转码。

## 准备工作

`.htaccess`文件用于在文件目录级别配置 Apache 服务器。配置也可以通过编辑`httpd.conf`来完成。因为许多服务器托管公司不允许访问安装了 Apache 的根目录，所以在这个例子中，我们使用`.htaccess`。这使得在目录级别进行服务器配置更容易，因为与主`httpd.conf`不同，它不需要服务器重新启动。创建或打开一个`.htaccess`文件。

## 如何做...

将以下代码添加到`.htaccess`文件中：

```html
<FilesMatch "\.(php|cgi|pl)$">
Header append Cache-Control "no-transform"
Header append Vary "User-Agent, Accept"
</FilesMatch>

```

将`.htaccess`文件上传到要应用规则的文件夹中。

通过这样做，我们已经防止了移动转码的发生。

## 它是如何工作的...

`FilesMatch`用于仅过滤 CGI 和 PHP 脚本，因为我们不希望将此规则应用于其他文件类型。

```html
<FilesMatch "\.(php|cgi|pl)$">

```

假设启用了 Apache 模块`mod_headers`，我们可以在`FilesMatch`部分中添加头部`Cache-Control "no-transform"`。

```html
Header append Cache-Control "no-transform"

```

## 还有更多...

以下资源可能有助于了解更多关于移动转码的信息。

### Microsoft Internet Information Server (IIS)

如果您正在使用**Microsoft Internet Information Server (IIS)**，可以使用软件界面进行配置。有关如何执行此操作的详细信息可以在以下位置找到：

[`mobiforge.com/developing/story/setting-http-headers-advise-transcoding-proxies`](http://mobiforge.com/developing/story/setting-http-headers-advise-transcoding-proxies)

### 负责任的重新格式化

以下文章提供了一些关于网络运营商进行的内容转码的影响的见解：

[`mobiforge.com/developing/blog/responsible-reformatting`](http://mobiforge.com/developing/blog/responsible-reformatting)

### MBP — Mobile Boilerplate

本章中使用的代码片段也包含在 Mobile Boilerplate 中：

[`github.com/h5bp/mobile-boilerplate/blob/master/.htaccess`](http://github.com/h5bp/mobile-boilerplate/blob/master/.htaccess)

# 添加移动 MIME 类型

目标浏览器：黑莓，塞班

黑莓和诺基亚浏览器支持许多移动专属内容类型。在本主题中，我们将看一些这些移动浏览器使用的 MIME 类型。由于服务器可能默认不识别它们，因此在服务器配置中正确添加它们非常重要。

## 准备工作

`.htaccess`文件用于在文件目录级别配置 Apache 服务器。它使得在目录级别进行服务器配置变得容易。创建或打开一个`.htaccess`文件。

## 如何做...

将以下代码添加到`.htaccess`文件中：

```html
AddType application/x-bb-appworld bbaw
AddType text/vnd.rim.location.xloc xloc
AddType text/x-vcard vcf
AddType application/octet-stream sisx
AddType application/vnd.symbian.install sis

```

将`.htaccess`文件上传到要应用规则的文件夹中。

## 它是如何工作的...

我们使用`AddType:`使移动 MIME 类型可识别：

| 代码 | 描述 |
| --- | --- |
| `AddType application/x-bb-appworld bbaw` | 包含在 BlackBerry App World™商店中找到的应用程序的应用程序 ID 的文本文件。 |
| `AddType text/vnd.rim.location.xloc xloc` | BlackBerry 地图位置文档。 |
| `AddType text/x-vcard vcf` | 一个 vCard 文件，一种用于电子名片的标准文件格式。 |
| `AddType application/octet-stream sisx` | 诺基亚类型 |
| `AddType application/vnd.symbian.install sis` | 诺基亚类型 |

## 还有更多...

有关 BlackBerry 支持的更多移动文件类型，请访问：

[`docs.blackberry.com/en/developers/deliverables/18169/index.jsp?name=Feature+and+Technical+Overview+-+BlackBerry+Browser6.0&language=English&userType=21&category=BlackBerry+Browser&subCategory=`](http://docs.blackberry.com/en/developers/deliverables/18169/index.jsp?name=Feature+and+Technical+Overview+-+BlackBerry+Browser6.0&language=English&userType=21&category=BlackBerry+Browser&subCategory=)

# 使缓存清单正确显示

目标浏览器：跨浏览器

如第六章中所解释的，*移动富媒体*，缓存清单用于离线 Web 应用程序。服务器可能无法识别此文件的扩展名。让我们看看如何添加正确的 MIME 类型。

## 准备工作

创建或打开一个`.htaccess`文件。

## 如何做...

添加以下代码：

```html
AddType text/cache-manifest appcache manifest

```

上传`.htaccess`文件到您希望应用规则的文件夹。

## 它是如何工作的...

缓存清单可以使用`.appcache`或`.manifest`作为其扩展名。通过将这两种类型都添加为`text/cache-manifest`，我们确保无论使用哪种类型，它们都可以正确呈现。

### MBP 移动样板

`.htaccess`规则包含在移动样板中：

[`github.com/h5bp/mobile-boilerplate/blob/master/.htaccess#L75`](http://github.com/h5bp/mobile-boilerplate/blob/master/.htaccess#L75)

# 设置远期过期标头

目标浏览器：跨浏览器

为文件设置远期过期标头可通过减少不必要的 HTTP 请求来提高站点性能。对于需要加载许多资源的富媒体站点，这可以提高整体性能。有不同的文件类型，根据文件的使用，我们选择不同的过期时间。

## 准备工作

创建或打开一个`.htaccess`文件。

## 如何做...

添加以下代码：

```html
<IfModule mod_expires.c>
ExpiresActive on
ExpiresDefault "access plus 1 month"
ExpiresByType text/cache-manifest "access plus 0 seconds"
ExpiresByType text/html "access plus 0 seconds"
ExpiresByType text/xml "access plus 0 seconds"
ExpiresByType application/xml "access plus 0 seconds"
ExpiresByType application/json "access plus 0 seconds"
ExpiresByType application/rss+xml "access plus 1 hour"
ExpiresByType image/x-icon "access plus 1 week"
ExpiresByType image/gif "access plus 1 month"
ExpiresByType image/png "access plus 1 month"
ExpiresByType image/jpg "access plus 1 month"
ExpiresByType image/jpeg "access plus 1 month"
ExpiresByType video/ogg "access plus 1 month"
ExpiresByType audio/ogg "access plus 1 month"
ExpiresByType video/mp4 "access plus 1 month"
ExpiresByType video/webm "access plus 1 month"
ExpiresByType text/x-component "access plus 1 month"
ExpiresByType font/truetype "access plus 1 month"
ExpiresByType font/opentype "access plus 1 month"
ExpiresByType application/x-font-woff "access plus 1 month"
ExpiresByType image/svg+xml "access plus 1 month"
ExpiresByType application/vnd.ms-fontobject "access plus 1 month"
ExpiresByType text/css "access plus 1 year"
ExpiresByType application/javascript "access plus 1 year"
ExpiresByType text/javascript "access plus 1 year"
<IfModule mod_headers.c>
Header append Cache-Control "public"
</IfModule>
</IfModule>

```

上传`.htaccess`文件到您希望应用规则的文件夹。

## 它是如何工作的...

以下是代码的分解，我们将看到它是如何工作的：

1.  白名单过期规则：

```html
ExpiresDefault "access plus 1 month"

```

1.  `cache.appcache`在 FF 3.6 中需要重新请求：

```html
ExpiresByType text/cache-manifest "access plus 0 seconds"

```

1.  您的文档 HTML 不应该被缓存：

```html
ExpiresByType text/html "access plus 0 seconds"

```

1.  数据不应该被缓存，因为它总是需要被拉取的：

```html
ExpiresByType text/xml "access plus 0 seconds"
ExpiresByType application/xml "access plus 0 seconds"
ExpiresByType application/json "access plus 0 seconds"

```

1.  RSS 订阅更新频率低于正常 API 数据：

```html
ExpiresByType application/rss+xml "access plus 1 hour"

```

1.  Favicon 不能被重命名，所以最好的方法是将其设置为一周后：

```html
ExpiresByType image/x-icon "access plus 1 week"

```

1.  对于诸如图像、视频和音频之类的大型媒体资源，我们可以将日期设置得更久远：

```html
ExpiresByType image/gif "access plus 1 month"
...
ExpiresByType video/webm "access plus 1 month"

```

1.  HTC 文件，如果您使用 HTML5 polyfill - CSS3PIE 会很有用：

```html
ExpiresByType text/x-component "access plus 1 month"

```

1.  安全地将 Webfonts 缓存一个月：

```html
ExpiresByType font/truetype "access plus 1 month"
...
ExpiresByType application/vnd.ms-fontobject "access plus 1 month"

```

1.  对于 CSS 和 JavaScript，我们可以将过期日期设置为一年后：

```html
ExpiresByType text/css "access plus 1 year"
ExpiresByType application/javascript "access plus 1 year"
ExpiresByType text/javascript "access plus 1 year"

```

### 还有更多...

这些都是相当远期的过期标头。它们假定您使用缓存破坏查询参数来控制版本：

```html
<script src="img/script_034543.js" ></script>

```

此外，考虑到过时的代理可能会错误缓存：

[`www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/`](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/)

#### 添加一个 Expires 或 Cache-Control 标头

在 Yahoo!开发者网络中，有关过期规则的解释非常好：

[`developer.yahoo.com/performance/rules.html#expires`](http://developer.yahoo.com/performance/rules.html#expires)

#### MBP 移动样板中的规则

这些规则包含在 Mobile Boilerplate 的`.htacess 文件中：`

[`github.com/h5bp/mobile-boilerplate/blob/master/.htaccess#L142`](http://github.com/h5bp/mobile-boilerplate/blob/master/.htaccess#L142)

# 使用 Gzip 压缩文件

目标浏览器：跨浏览器

前端开发人员在决定如何减少在网络上传输 HTTP 请求和响应所需的时间方面发挥着重要作用。Gzip 压缩可通过减小 HTTP 响应的大小来减少响应时间。

Gzip 可以大大减小响应大小，通常可减小 70%。Gzip 在现代浏览器中得到广泛支持。

大多数服务器默认只压缩某些文件类型，因此最好定义支持广泛的文本文件的规则，包括 HTML、XML 和 JSON。

## 准备工作

创建或打开一个 `.htaccess` 文件。

## 如何做...

将以下代码添加到 `.htaccess` 中：

```html
<IfModule mod_deflate.c>
<IfModule mod_setenvif.c>
<IfModule mod_headers.c>
SetEnvIfNoCase ^(Accept-EncodXng|X-cept-Encoding|X{15}|~{15}|-{15})$ ^((gzip|deflate)\s,?\s(gzip|deflate)?|X{4,13}|~{4,13}|-{4,13})$ HAVE_Accept-Encoding
RequestHeader append Accept-Encoding "gzip,deflate" env=HAVE_Accept-Encoding
</IfModule>
</IfModule>
<IfModule filter_module>
FilterDeclare COMPRESS
FilterProvider COMPRESS DEFLATE resp=Content-Type $text/html
FilterProvider COMPRESS DEFLATE resp=Content-Type $text/css
FilterProvider COMPRESS DEFLATE resp=Content-Type $text/javascript
FilterProvider COMPRESS DEFLATE resp=Content-Type $text/plain
FilterProvider COMPRESS DEFLATE resp=Content-Type $text/xml
FilterProvider COMPRESS DEFLATE resp=Content-Type $text/x-component
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/javascript
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/json
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/xml
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/x-javascript
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/xhtml+xml
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/rss+xml
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/atom+xml
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/vnd.ms-fontobject
FilterProvider COMPRESS DEFLATE resp=Content-Type $image/svg+xml
FilterProvider COMPRESS DEFLATE resp=Content-Type $application/x-font-ttf
FilterProvider COMPRESS DEFLATE resp=Content-Type $font/opentype
FilterChain COMPRESS
FilterProtocol COMPRESS DEFLATE change=yes;byteranges=no
</IfModule>
<IfModule !mod_filter.c>
AddOutputFilterByType DEFLATE text/html text/plain text/css application/json
AddOutputFilterByType DEFLATE text/javascript application/javascript application/x-javascript
AddOutputFilterByType DEFLATE text/xml application/xml text/x-component
AddOutputFilterByType DEFLATE application/xhtml+xml application/rss+xml application/atom+xml
AddOutputFilterByType DEFLATE image/svg+xml application/vnd.ms-fontobject application/x-font-ttf font/opentype
</IfModule>
</IfModule>

```

将 `.htaccess` 文件上传到要应用规则的文件夹中。

## 它是如何工作的...

以下代码强制对损坏的标头进行通缩，以便检测损坏的模式，`mod_setenvif` 用于执行正则表达式匹配并设置一个环境变量，指示损坏的 Accept-Encoding 标头存在：

```html
SetEnvIfNoCase ^(Accept-EncodXng|X-cept-Encoding|X{15}|~{15}|-{15})$ ^((gzip|deflate)\s,?\s(gzip|deflate)?|X{4,13}|~{4,13}|-{4,13})$ HAVE_Accept-Encoding

```

强制请求标头支持压缩很简单：

```html
RequestHeader append Accept-Encoding "gzip,deflate" env=HAVE_Accept-Encoding

```

压缩 HTML、TXT、CSS、JavaScript、JSON、XML、HTC：

```html
<IfModule filter_module>
FilterDeclare COMPRESS
...
FilterProtocol COMPRESS DEFLATE change=yes;byteranges=no
</IfModule>

```

对于 Apache 2.1 之前的旧版本：

```html
<IfModule !mod_filter.c>
AddOutputFilterByType DEFLATE text/html text/plain text/css application/json
AddOutputFilterByType DEFLATE text/javascript application/javascript application/x-javascript
AddOutputFilterByType DEFLATE text/xml application/xml text/x-component
AddOutputFilterByType DEFLATE application/xhtml+xml application/rss+xml application/atom+xml
AddOutputFilterByType DEFLATE image/svg+xml application/vnd.ms-fontobject application/x-font-ttf font/opentype
</IfModule>

```

## 还有更多...

需要注意的是，图像和 PDF 文件不需要进行 Gzip 压缩，因为它们已经默认进行了压缩。对它们进行 Gzip 压缩将浪费 CPU 使用率，甚至会增加文件大小。

### 超越 Gzip

*Marcel Duran* 在 Yahoo! Network 上的一篇关于 Gzip 的文章谈到了最近的研究和服务器端方法：

[`developer.yahoo.com/blogs/ydn/posts/2010/12/pushing-beyond-gzipping/`](http://developer.yahoo.com/blogs/ydn/posts/2010/12/pushing-beyond-gzipping/)

# 移除 ETags

目标浏览器：跨浏览器

ETags 代表 **实体标签**。实体是诸如 CSS 或 JavaScript 文件、图像等组件。实体标签的作用是标识组件的特定版本。您可以在 *Yahoo! Developer Network, 高性能网站：规则 13 配置 ETags* ([`developer.yahoo.com/blogs/ydn/posts/2007/07/high_performanc_11/`](http://developer.yahoo.com/blogs/ydn/posts/2007/07/high_performanc_11/)) 中找到更多详细信息。

如果您的网站由多个服务器托管，例如在内容交付网络上，ETag 的验证机制可能会导致额外的重新获取。验证模型中几乎没有优势，所以最佳实践是只需移除 ETag。

## 准备工作

创建或打开一个 `.htaccess` 文件。

## 如何做...

添加以下代码：

```html
<IfModule mod_headers.c>
Header unset Etag
</IfModule>
FileETag None

```

将 `.htaccess` 文件上传到要应用规则的文件夹中。

## 它是如何工作的...

首先，我们取消当前已配置文件的 ETag：

```html
<IfModule mod_headers.c>
Header unset Etag
</IfModule>

```

其次，我们使用 `FileTag None` 来确保文件的 ETag 被移除：

```html
FileETag None

```

## 还有更多...

以下部分提供了有关 ETags 的更多信息供您参考。

### 在 IIS 服务器上同步 ETag 值

如果您正在运行 IIS 服务器，为了解决问题，您必须同步运行 IIS 5.0 的所有 Web 服务器上的 ETag 值。为此，请使用 `Mdutil.exe` 工具从其中一个 Web 服务器检索 ETag 值。然后，在所有其他 Web 服务器上设置相同的 ETag 值。

更详细的说明可以在以下 Microsoft 支持文章中找到：

[`support.microsoft.com/?id=922733`](http://support.microsoft.com/?id=922733)

### 高性能网站

*Steve Souders* 在他的 *高性能网站* 系列中解释了配置规则：

*高性能网站：规则 13* — *配置 ETags：*

[`developer.yahoo.com/blogs/ydn/posts/2007/07/high_performanc_11/`](http://developer.yahoo.com/blogs/ydn/posts/2007/07/high_performanc_11/)

### David Walsh 博客

*David Walsh*的博客网站包含了 Eric Wendelin 的一篇文章 - *使用.htaccess 改善您的 YSlow 等级*，其中也提到了这个配方中解决的问题：

[`davidwalsh.name/yslow-htaccess`](http://davidwalsh.name/yslow-htaccess)

### MBP - 移动样板

实体标签的移除也包含在移动样板中：

[`github.com/h5bp/mobile-boilerplate/blob/master/.htaccess#L211-L218`](http://github.com/h5bp/mobile-boilerplate/blob/master/.htaccess#L211-L218)
