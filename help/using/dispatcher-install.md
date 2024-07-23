---
title: 安装 Dispatcher
description: 了解如何在 Microsoft&reg; Internet Information Server、Apache Web Server 和 Sun Java&trade; Web Server-iPlanet 上安装 Dispatcher 模块。
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: 9be9f5935c21ebbf211b5da52280a31772993c2e
workflow-type: tm+mt
source-wordcount: '3748'
ht-degree: 100%

---

# 安装 Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

使用 [Dispatcher 发行说明](release-notes.md)页面可获取适用于您的操作系统和 Web 服务器的最新 Dispatcher 安装文件。Dispatcher 版本号独立于 Adobe Experience Manager 版本号，并且与 Adobe Experience Manager 6.x、5.x 和 Adobe CQ 5.x 版本兼容。

>[!NOTE]
>
>Adobe Experience Manager 6.5 要求 Dispatcher 版本 4.3.2 或更高版本。也就是说，Dispatcher 版本独立于 AEM，例如 Dispatcher 版本 4.3.2 也与 Adobe Experience Manager 6.4 兼容。

使用以下文件命名约定：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

例如，`dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` 文件包含用于 Apache 2.4 网络服务器（该服务器在 Linux® i686 运行）的 Dispatcher 4.3.1 版本，并使用 **tar** 格式打包。

下表列出了在每个 Web 服务器的文件名中使用的 Web 服务器标识符：

| Web 服务器 | 安装套件 |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;其他参数> |
| Microsoft® Internet Information Server 7.5, 8, 8.5, 10 | dispatcher-**iis**-&lt;其他参数> |
| Sun Java™ Web Server iPlanet | dispatcher-**ns**-&lt;其他参数> |

>[!CAUTION]
>
>请确保安装了适用于您平台的最新版本 Dispatcher。每年都应将您的 Dispatcher 实例升级到最新版本以使用产品改进功能。

>[!NOTE]
>
>专门从 4.3.3 版升级到 4.3.4 版的客户都会发现为不可缓存内容设置缓存标头的方式行为有所不同。若要详细了解此更改，请阅读[发行说明](/help/using/release-notes.md#nov)页。

每个存档都包含以下文件：

* Dispatcher 模块
* 示例配置文件
* 包含安装说明和最新信息的自述文件
* 列出当前版本或过去版本中修复的问题的变更文件

>[!NOTE]
>
>在开始安装之前，查看自述文件，了解是否有任何最新的更改/特定于平台的说明。

<!-- 

Comment Type: draft

<h3>Supported Web Servers</h3>

 -->

<!-- 

Comment Type: draft

<p>The following web servers are supported for use with Dispatcher version 4.1.12:</p>

 -->

<!-- 

Comment Type: draft

<p>The following sections detail the specific Web server installation procedures.</p>

 -->

## Microsoft® Internet Information Server {#microsoft-internet-information-server}

有关如何安装此 Web 服务器的信息，请参阅以下资源：

* Microsoft® 自带的有关 Internet Information Server 的文档
* [&quot;The Official Microsoft® IIS site&quot;](https://www.iis.net/)

### 必需的 IIS 组件 {#required-iis-components}

IIS 版本 8.5 和 10 要求安装以下 IIS 组件：

* ISAPI 扩展

您还必须安装 Web Server (IIS) 角色。使用 Server Manager 添加角色和组件。

## Microsoft® IIS - 安装 Dispatcher 模块 {#microsoft-iis-installing-the-dispatcher-module}

Microsoft® Internet Information System 的必需存档为：

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP 文件包含以下文件：

| 文件 | 描述 |
|--- |--- |
| `disp_iis.dll` | Dispatcher 动态链接库文件。 |
| `disp_iis.ini` | IIS 的配置文件。可根据您的要求更新此示例。**注意**：ini 文件必须具有与 dll 相同的命名根。 |
| `dispatcher.any` | Dispatcher 的示例配置文件。 |
| `author_dispatcher.any` | 使用创作实例的 Dispatcher 的示例配置文件。 |
| 自述文件 | 包含安装说明和最新信息的自述文件。**注意**：先查看此文件，然后再开始安装。 |
| 变更文件 | 列出当前版本或过去版本中修复的问题的变更文件。 |

使用以下过程可将 Dispatcher 文件复制到正确位置。

1. 使用 Windows 资源管理器创建 `<IIS_INSTALLDIR>/Scripts` 目录，例如 `C:\inetpub\Scripts`。

1. 将 Dispatcher 包中的以下文件提取到此 Scripts 目录中：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 下列文件之一，取决于 Dispatcher 是使用 AEM 创作实例还是发布实例：
      * 创作实例：`author_dispatcher.any`
      * 发布实例：`dispatcher.any`

## Microsoft® IIS - 配置 Dispatcher INI 文件 {#microsoft-iis-configure-the-dispatcher-ini-file}

要配置Dispatcher安装，请编辑该 `disp_iis.ini` 文件。`.ini` 文件的基本格式如下所示：

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

下表描述了每个属性。

| 参数 | 描述 |
|--- |--- |
| `configpath` | `dispatcher.any` 在本地文件系统中的位置（绝对路径）。 |
| `logfile` | `dispatcher.log` 文件的位置。如果未设置此位置，则日志消息将转到 Windows 事件日志。 |
| `loglevel` | 定义用于将消息输出到事件日志的日志级别。可以在日志级别为日志文件指定以下值：<br/>0 - 仅错误消息。<br/>1 - 错误和警告。<br/>2 - 错误、警告、以及信息性消息 <br/>3 - 错误、警告和信息性消息和调试消息。<br/>**注意**：在安装和测试期间将日志级别设置为 3，在生产环境中运行时将日志级别设置为 0。 |
| `replaceauthorization` | 指定如何处理 HTTP 请求中的授权标头。以下值有效：<br/>0 - 未修改授权标头。<br/>1 - 将除“Basic”以外的任何名为“Authorization”的标头替换为其 `Basic <IIS:LOGON\_USER>` 等效标头。<br/> |
| `servervariables` | 定义如何处理服务器变量。<br/>0 - IIS 服务器变量不会发送到 Dispatcher 或 AEM。<br/>1 - 所有 IIS 服务器变量（例如 `LOGON\_USER, QUERY\_STRING, ...`）都与请求标头一起发送到 Dispatcher（如果未缓存，还将发送到 AEM 实例）。<br/>服务器变量包括 `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` 及其他。请参阅 IIS 文档以查看完整的变量列表以及详细信息。 |
| `enable_chunked_transfer` | 定义是为客户端响应启用 (1) 还是禁用 (0) 块传输。默认值为 0。 |

示例配置：

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### 配置 Microsoft® IIS {#configuring-microsoft-iis}

配置 IIS 以集成 Dispatcher ISAPI 模块。在 IIS 中，使用通配符应用程序映射。

### 配置匿名访问 - IIS 8.5 和 10 {#configuring-anonymous-access-iis-and}

创作实例上的默认 Flush 复制代理已配置为不随刷新请求发送安全凭据。因此，您使用 Dispatcher 缓存的网站必须允许匿名访问。

如果您的网站使用身份验证方法，则必须相应地配置 Flush 复制代理。

1. 打开 IIS Manager 并选择要用作 Dispatcher 缓存的网站。
1. 使用“功能视图”模式，在 IIS 部分中双击“身份验证”。
1. 如果未启用“匿名身份验证”，请选择“匿名身份验证”，并在“操作”区域中，单击“启用”。

### 集成 Dispatcher ISAPI 模块 - IIS 8.5 和 10 {#integrating-the-dispatcher-isapi-module-iis-and}

使用以下过程可将 Dispatcher ISAPI 模块添加到 IIS。

1. 打开 IIS Manager。
1. 选择您将用作 Dispatcher 缓存的网站。
1. 使用“功能视图”模式，在 IIS 部分中双击“处理程序映射”。
1. 在“处理程序映射”页面的“操作”面板中，单击“添加通配符脚本映射”，添加以下属性值，然后单击“确定”：

   * 请求路径：&#42;
   * 可执行文件：disp_iis.dll 文件的绝对路径，例如 `C:\inetpub\Scripts\disp_iis.dll`。
   * 名称：处理程序映射的描述性名称，例如 `Dispatcher`。

1. 在出现的对话框中，要将 disp_iis.dll 库添加到 ISAPI 和 CGI 限制列表，请单击&#x200B;**是**。

   对于 IIS 7.0 和 7.5，配置是完整的。如果您配置的是 IIS 8.0，请使用剩余步骤进行配置。

1. (IIS 8.0) 在处理程序映射列表中，选择您创建的映射，然后在“操作”区域中单击“编辑”。
1. (IIS 8.0) 在“编辑脚本映射”对话框中，单击“请求限制”按钮。
1. (IIS 8.0) 要确保处理程序用于尚未缓存的文件和文件夹，请取消选择&#x200B;**仅在映射到请求时调用处理程序**。单击&#x200B;**确定**。
1. (IIS 8.0) 在“编辑脚本映射”对话框中，单击“确定”。

### 配置对缓存的匿名访问 - IIS 8.5 和 10 {#configuring-access-to-the-cache-iis-and}

向默认 App Pool 用户提供对将用作 Dispatcher 缓存的文件夹的写访问权限。

1. 右键单击将用作 Dispatcher 缓存的网站的根文件夹，然后单击“属性”，例如 `C:\inetpub\wwwroot`。
1. 在“安全性”选项卡上，单击“编辑”，然后在“权限”对话框中，单击“添加”。这将打开一个对话框，可在其中选择用户帐户。单击“位置”按钮，选择您的计算机名称，然后单击“确定”。

   将此对话框保持打开状态，同时完成下一个步骤。

1. 在 IIS Manager 中，选择要用于 Dispatcher 缓存的 IIS 站点，然后在窗口右侧单击“高级设置”。
1. 选择应用程序池属性的值并将该值复制到剪贴板。
1. 返回打开的对话框。在“输入要选择的对象名称”框中，键入 `IIS AppPool\`，然后粘贴剪贴板的内容。该值应与以下示例类似：

   `IIS AppPool\DefaultAppPool`

1. 单击“检查名称”按钮。在 Windows 解析用户帐户时，单击“确定”。
1. 在 Dispatcher 文件夹的“权限”对话框中，选择您刚刚添加的帐户，启用该帐户的所有权限（**完全控制权限除外**），然后单击“确定”。单击“确定”可关闭“文件夹属性”对话框。

### 注册 JSON Mime 类型 - IIS 8.5 和 10 {#registering-the-json-mime-type-iis-and}

如果您希望 Dispatcher 允许 JSON 调用，可使用以下过程注册 JSON MIME 类型。

1. 在 IIS Manager 中，选择您的网站并使用“功能视图”，双击“Mime 类型”。
1. 如果 JSON 扩展未在列表中，请在“操作”面板中单击“添加”，输入以下属性值，然后单击“确定”：

   * 文件扩展名：`.json`
   * MIME 类型：`application/json`

### 删除 bin 隐藏部分 - IIS 8.5 和 10 {#removing-the-bin-hidden-segment-iis-and}

使用以下过程可删除 `bin` 隐藏部分。非新网站可能包含此隐藏部分。

1. 在 IIS Manager 中，选择您的网站并使用“功能视图”，双击“请求过滤”。
1. 选择 `bin` 部分，单击“移除”，然后在确认对话框中，单击“是”。

### 将 IIS 消息记录到文件 - IIS 8.5 和 10 {#logging-iis-messages-to-a-file-iis-and}

使用以下过程可将 Dispatcher 日志消息写入日志文件而非 Windows 事件日志。配置 Dispatcher 以使用日志文件，并向 IIS 提供对该文件的写访问权限。

1. 使用 Windows 资源管理器在 IIS 安装的日志文件夹下创建名为 `dispatcher` 的文件夹。典型安装的此文件夹的路径为 `C:\inetpub\logs\dispatcher`。

1. 右键单击 Dispatcher 文件夹，然后单击&#x200B;**属性**。
1. 在“安全”选项卡上，单击&#x200B;**编辑**。
1. 在“权限”对话框中，单击&#x200B;**添加**。这将打开一个对话框，可在其中选择用户帐户。单击“位置”按钮，选择您的计算机名称，然后单击“确定”。

   将此对话框保持打开状态，同时完成下一个步骤。

1. 在 IIS Manager 中，选择要用于 Dispatcher 缓存的 IIS 站点，然后在窗口右侧单击“高级设置”。
1. 选择应用程序池属性的值并将该值复制到剪贴板。
1. 返回打开的对话框。在“输入要选择的对象名称”框中，键入 `IIS AppPool\`，然后粘贴剪贴板的内容。该值应与以下示例类似：

   `IIS AppPool\DefaultAppPool`

1. 单击“检查名称”按钮。在 Windows 解析用户帐户时，单击“确定”。
1. 在 Dispatcher 文件夹的“权限”对话框中，选择您刚刚添加的帐户，启用该帐户的所有权限（**完全控制权限除外），** 然后单击“确定”。单击“确定”可关闭“文件夹属性”对话框。
1. 使用文本编辑器打开 `disp_iis.ini` 文件。
1. 要配置日志文件的位置，添加一行类似于以下示例的文本，然后保存该文件：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### 后续步骤 {#next-steps}

您必须了解以下内容，然后才能开始使用 Dispatcher：

* [配置](dispatcher-configuration.md) Dispatcher
* [配置 AEM](page-invalidate.md) 以使用 Dispatcher。

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>此处介绍了 **Windows** 和 **UNIX®** 下的安装说明。务必小心执行这些步骤。

### 安装 Apache Web Server {#installing-apache-web-server}

有关如何安装 Apache Web Server 的信息，请参阅安装手册 - [在线版本](https://httpd.apache.org/)或分发版。

>[!CAUTION]
>
>如果通过编译源文件来创建 Apache 二进制文件，请确保打开 **`dynamic modules support`**。启用此选项可使用任意 **--enable-shared** 选项来执行此操作。最低要求是包含 `mod_so` 模块。
>
>有关更多信息，请参阅 Apache Web Server 安装手册。

另请参阅 Apache HTTP Server [安全提示](https://httpd.apache.org/docs/2.4/misc/security_tips.html) 和[安全报告](https://httpd.apache.org/security_report.html)。

### Apache Web Server - 添加 Dispatcher 模块 {#apache-web-server-add-the-dispatcher-module}

Dispatcher 的提供形式为：

* **Windows**：动态链接库 (DLL)
* **UNIX®**：动态共享对象 (DSO)

安装存档文件包含以下文件 - 取决于您选择的是 Windows 还是 UNIX®：

| 文件 | 描述 |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows：Dispatcher 动态链接库文件。 |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>.so | UNIX®：Dispatcher 共享对象库文件。 |
| mod_dispatcher.so | UNIX®：示例链接。 |
| http.conf.disp&lt;x> | Apache Server 的示例配置文件。 |
| dispatcher.any | Dispatcher 的示例配置文件。 |
| 自述文件 | 包含安装说明和最新信息的自述文件。**注意**：先查看此文件，然后再开始安装。 |
| 变更文件 | 列出当前版本或过去版本中修复的问题的变更文件。 |

使用以下步骤可将 Dispatcher 添加到 Apache Web Server：

1. 将 Dispatcher 文件放置到适当的 Apache 模块目录中：

   * **Windows**：放置 `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **UNIX®**：根据安装找到 `<APACHE_ROOT>/libexec` 或 `<APACHE_ROOT>/modules` 目录。\
     将 `dispatcher-apache<options>.so` 复制到此目录中。\
     要简化长期维护，您还可以创建一个指向 Dispatcher 的名为 `mod_dispatcher.so` 的符号链接：\
     `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. 将 dispatcher.any 文件复制到 `<APACHE_ROOT>/conf` 目录。

   **注意：**&#x200B;您可以将此文件放置到不同的位置，前提是相应地配置 Dispatcher 模块的 DispatcherLog 属性。（请参阅以下特定于 Dispatcher 的配置条目。）

### Apache Web Server - 配置 SELinux 属性 {#apache-web-server-configure-selinux-properties}

如果您在支持 SELinux 的 Red Hat® Linux® Kernel 2.6 上运行 Dispatcher，您可能会在 Dispatcher 日志文件中看到类似的错误消息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

此错误可能是由于启用的 SELinux 安全性导致的。如果是这样，请执行以下操作：

* 配置 Dispatcher 模块文件的 SELinux 上下文。
* 启用 HTTPD 脚本和模块以建立网络连接。
* 配置存储了缓存文件的 docroot 的 SELinux 上下文。

在终端窗口中输入以下命令，将 `[path to the dispatcher.so file]` 替换为已安装到 Apache Web Server 的 Dispatcher 模块的路径，并将 *`path to the docroot`* 替换为 docroot 的路径（例如 `/opt/cq/cache`）：

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_rw_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_rw_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - 为 Dispatcher 配置 Apache Web Server {#apache-web-server-configure-apache-web-server-for-dispatcher}

必须使用 `httpd.conf` 配置 Apache Web Server。在 Dispatcher 安装套件中，找到一个名为 `httpd.conf.disp<x>` 的示例配置文件。

必须执行以下步骤：

1. 导航到 `<APACHE_ROOT>/conf`。
1. 打开 `httpd.conf` 以进行编辑。
1. 必须按列出的顺序添加以下配置条目：

   * **LoadModule**，用于启动时加载模块。
   * 特定于 Dispatcher 的配置条目，包括 **DispatcherConfig、DispatcherLog** 和 **DispatcherLogLevel**。
   * **SetHandler**，用于激活 Dispatcher。**LoadModule**。
   * **ModMimeUsePathInfo**，用于配置 **mod_mime** 的行为。

1. （可选）建议您更改 htdocs 目录的所有者：

   * Apache 服务器 以 root 身份启动，但子进程作为守护进程启动（出于安全目的）。DocumentRoot (`<APACHE_ROOT>/htdocs`) 必须属于用户守护进程：

     ```xml
     cd <APACHE_ROOT>  
     chown -R daemon:daemon htdocs
     ```

**LoadModule**

下表列出了可以使用的示例；确切的条目取决于您的特定 Apache Web Server：

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| UNIX®（使用符号链接） | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>必须完全按照上述示例编写每个语句的第一个参数。
>
>有关此命令的完整详细信息，请参阅提供的示例配置文件和 Apache Web Server 文档。

**特定于 Dispatcher 的配置条目**

特定于 Dispatcher 的配置条目将置于 LoadModule 条目之后。下表列出了适用于 UNIX® 和 Windows 的示例配置：

**Windows &amp; UNIX®**

```
...
<IfModule disp_apache2.c>
DispatcherConfig conf/dispatcher.any
DispatcherLog logs/dispatcher.log DispatcherLogLevel 3
DispatcherNoServerHeader 0 DispatcherDeclineRoot 0
DispatcherUseProcessedURL 0
DispatcherPassError 0
DispatcherKeepAliveTimeout 60
</IfModule>
...
```

>[!NOTE]
>
>专门从 4.3.3 版升级到 4.3.4 版的客户都会发现为不可缓存内容设置缓存标头的方式行为有所不同。若要详细了解此更改，请阅读[发行说明](/help/using/release-notes.md#nov)页。

单个配置参数：

| 参数 | 描述 |
|--- |--- |
| DispatcherConfig | Dispatcher 配置文件的位置和名称。<br/>当此属性位于主服务器配置中时，所有虚拟主机都将会继承属性值。不过，虚拟主机可以包括 DispatcherConfig 属性来覆盖主服务器配置。 |
| DispatcherLog | 日志文件的位置和名称。 |
| DispatcherLogLevel | 日志文件的日志级别：<br/>0 - 错误 <br/>1 - 警告 <br/>2 - 信息 <br/>3 - 调试 <br/>**注意**：在安装和测试期间将日志级别设置为 3，在生产环境中运行时将日志级别设置为 0。 |
| DispatcherNoServerHeader | *此参数已弃用且无效。*<br/><br/> 定义要使用的服务器标头：<br/><ul><li>未定义或 0 - HTTP 服务器标头包含 AEM 版本。 </li><li>1 - 使用 Apache 服务器标头。</li></ul> |
| DispatcherDeclineRoot | 定义是否拒绝对根“/”的请求：<br/>**0** - 接受对 / 的请求 <br/>**1** - Dispatcher 不处理对 / 的请求。相反，使用 mod_alias 进行正确的映射。 |
| DispatcherUseProcessedURL | 定义是否将预处理的 URL 用于 Dispatcher 执行的所有进一步处理：<br/>**0** - 使用传递给 Web 服务器的原始 URL。<br/>**1** - Dispatcher 使用已由其前面的处理程序处理的 URL（即 `mod_rewrite`），而不是使用传递给 网络服务器的原始 URL。例如，原始 URL 或处理后的 URL 与 Dispatcher 过滤器匹配。此 URL 也用作缓存文件结构的基础。有关 mod_rewrite 的信息，请参阅 Apache 网站文档；例如，Apache 2.4。使用 mod_rewrite 时，使用标志“passthrough”（通过下一个处理程序）强制重写引擎将内部 request_rec 结构的 URI 字段设置为 filename 字段的值。 |
| DispatcherPassError | 定义如何支持 ErrorDocument 处理的错误代码：<br/>**0** - Dispatcher 将所有错误响应假脱机到客户端。<br/>**1** - Dispatcher 没有将错误响应发送给客户端（状态代码大于或等于 400）。相反，它将状态代码传递给 Apache，Apache 允许 ErrorDocument 指令处理此类状态代码。<br/>**代码范围** - 指定将其响应传递到 Apache 的错误代码的范围。其他错误代码将传递到客户端。例如，以下配置将错误 412 的响应传递到客户端，并将所有其他错误（DispatcherPassError 400-411、413-417）的响应传递到 Apache。 |
| DispatcherKeepAliveTimeout | 指定保持活动状态超时时间（以秒为单位）。从 Dispatcher 版本 4.2.0 开始，默认的保持活动状态值为 60。如果值为 0，则禁用保持活动状态。 |
| DispatcherNoCanonURL | 将此参数设置为“启用”会将原始 URL 而不是规范化的 URL 传递到后端，并且将覆盖 DispatcherUseProcessedURL 的设置。默认值为“禁用”。<br/>**注意**：Dispatcher 配置中的过滤器规则总是根据经过处理的 URL 而不是原始 URL 进行评估。 |

>[!NOTE]
>
>路径条目相对于 Apache Web Server 的根目录。

>[!NOTE]
>
>服务器标头的默认设置为：
>
>`ServerTokens Full`
>
>`DispatcherNoServerHeader 0`
>
>其中显示了 AEM 版本（用于统计目的）。如果您想在标头中禁止显示此类信息，可以设置以下内容：
>
>`ServerTokens Prod`
>
>有关更多信息，请参阅[有关 ServerTokens 指令的 Apache 文档（例如，适用于 Apache 2.4 的文档）](https://httpd.apache.org/docs/2.4/mod/core.html)。

**SetHandler**

在这些条目之后，您必须将 **SetHandler** 语句添加到配置的上下文（`<Directory>`、`<Location>`）中以便 Dispatcher 处理传入请求。以下示例将 Dispatcher 配置为处理对整个网站的请求：

**Windows 和 Unix®**

```
...  
<Directory />  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

以下示例将 Dispatcher 配置为处理对虚拟域的请求：

**Windows**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot _\[cache-path\]_\\docs  
<Directory _\[cache-path\]_\\docs>  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

**UNIX®**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot /usr/apachecache/docs  
<Directory /usr/apachecache/docs>  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

>[!NOTE]
>
>必须&#x200B;*完全按照上述示例*&#x200B;编写 **SetHandler** 语句的参数，因为这是模块中所定义处理程序的名称。
>
>有关此命令的完整详细信息，请参阅提供的示例配置文件和 Apache Web Server 文档。

**ModMimeUsePathInfo**

在 **SetHandler** 语句之后，您还应添加 **ModMimeUsePathInfo** 定义。

>[!NOTE]
>
>应仅在使用 Dispatcher 4.0.9 或更高版本时使用和配置 `ModMimeUsePathInfo` 参数。
>
>请注意，Dispatcher 4.0.9 版本已于 2011 年发布。如果您使用的是旧版本，则应升级到最新的 Dispatcher 版本。

应为所有 Apache 配置将 **ModMimeUsePathInfo** 参数设置为 `On`：

`ModMimeUsePathInfo On`

mod_mime 模块（例如 [Apache 模块 mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)）用于将内容元数据分配给为 HTTP 响应选择的内容。默认设置表示 `mod_mime` 确定内容类型。因此，只考虑映射到文件或目录的 URL 部分。

在设置为 `On` 时，`ModMimeUsePathInfo` 参数会指定 `mod_mime` 根据&#x200B;*完整* URL 确定内容类型；这意味着虚拟资源会根据其扩展应用元信息。

以下示例激活 **ModMimeUsePathInfo**：

**Windows 和 Unix®**

```
...  
<Directory />  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
ModMimeUsePathInfo On  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

### 启用对 HTTPS 的支持（UNIX® and Linux®）{#enable-support-for-https-unix-and-linux}

Dispatcher 使用 OpenSSL 实现 HTTP 上的安全通信。从 Dispatcher 版本 **4.2.0** 开始，支持 OpenSSL 1.0.0 和 OpenSSL 1.0.1。默认情况下，Dispatcher 使用 OpenSSL 1.0.0。要使用 OpenSSL 1.0.1，请使用以下过程创建符号链接，以便 Dispatcher 使用已安装的 OpenSSL 库。

1. 打开终端，并将当前目录更改为安装了 OpenSSL 库的目录，例如：

   ```shell
   cd /usr/lib64
   ```

1. 要创建符号链接，请输入以下命令：

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
>
>如果您使用的是自定义版本的 Apache，请确保使用相同版本的 [OpenSSL](https://www.openssl.org/source/) 编译 Apache 和 Dispatcher。

### 后续步骤 {#next-steps-1}

您必须了解以下内容，然后才能开始使用 Dispatcher：

* [配置](dispatcher-configuration.md) Dispatcher
* [配置 AEM](page-invalidate.md) 以使用 Dispatcher。

## Sun Java™ System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
>
>此处介绍了 Windows 和 UNIX® 环境的说明。
>
>务必小心选择要执行的操作。

### Sun Java™ System Web Server/iPlanet - 安装您的 Web 服务器 {#sun-java-system-web-server-iplanet-installing-your-web-server}

有关如何安装这些 Web 服务器的完整信息，参阅其各自的文档：

* Sun Java™ System Web Server
* iPlanet Web Server

### Sun Java™ System Web Server/iPlanet - 添加 Dispatcher 模块 {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher 的提供形式为：

* **Windows**：动态链接库 (DLL)
* **UNIX®**：动态共享对象 (DSO)

安装存档文件包含以下文件 - 取决于您选择的是 Windows 还是 UNIX®：

| 文件 | 描述 |
|---|---|
| `disp_ns.dll` | Windows：Dispatcher 动态链接库文件。 |
| `dispatcher.so` | UNIX®：Dispatcher 共享对象库文件。 |
| `dispatcher.so` | UNIX®：示例链接。 |
| `obj.conf.disp` | iPlanet/Sun Java™ System Web Server 的示例配置文件。 |
| `dispatcher.any` | Dispatcher 的示例配置文件。 |
| 自述文件 | 包含安装说明和最新信息的自述文件。**注意：** 先查看此文件，然后再开始安装。 |
| 变更文件 | 列出当前版本或过去版本中修复的问题的变更文件。 |

使用以下步骤可将 Dispatcher 添加到您的 Web 服务器：

1. 将 Dispatcher 文件放置到 Web 服务器的 `plugin` 目录中：

### Sun Java™ System Web Server/iPlanet - 针对 Dispatcher 进行配置 {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

必须使用 `obj.conf` 配置网络服务器。在 Dispatcher 安装套件中，找到名为 `obj.conf.disp` 的示例配置文件。

1. 导航到 `<WEBSERVER_ROOT>/config`。
1. 打开 `obj.conf` 以进行编辑。
1. 复制以下行：\
   `Service fn="dispService"`\
   从 `obj.conf.disp` 到 `obj.conf` 的初始化部分。

1. 保存更改。
1. 打开 `magnus.conf` 以进行编辑。
1. 复制以下两行：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   从 `obj.conf.disp` 到 `magnus.conf` 的初始化部分。

1. 保存更改。

>[!NOTE]
>
>以下配置都应该在一行上。此外，`$(SERVER_ROOT)` 和 `$(PRODUCT_SUBDIR)` 必须替换为各自的值。

**Init**

下表列出了可以使用的示例；确切的条目取决于您的特定 Web 服务器：

**Windows 和 Unix®**

```
...  
Init funcs="dispService,dispInit" fn="load-modules" shlib="$(SERVER\_ROOT)/plugins/dispatcher.so"  
Init fn="dispInit" config="$(PRODUCT\_SUBDIR)/dispatcher.any" loglevel="1" logfile="$(PRODUCT\_SUBDIR)/logs/dispatcher.log"  
keepalivetimeout="60"  
...
```

其中：

| 参数 | 描述 |
|--- |--- |
| `config` | 配置文件 `dispatcher.any.` 的位置和名称。 |
| `logfile` | 日志文件的位置和名称。 |
| `loglevel` | 在将消息写入日志文件时的日志级别：<br/>**0** 错误 <br/>**1** 警告 <br/>**2** 信息 <br/>**3** 调试 <br/>**注意：**&#x200B;在安装和测试期间将日志级别设置为 3，在生产环境中运行时将日志级别设置为 0。 |
| `keepalivetimeout` | 指定保持活动状态超时时间（以秒为单位）。从 Dispatcher 版本 4.2.0 开始，默认的保持活动状态值为 60。如果值为 0，则禁用保持活动状态。 |

根据您的要求，可以将 Dispatcher 定义为对象的服务。要为整个网站配置 Dispatcher，请编辑默认对象：

**Windows**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)\\dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*\\\*"  
...
```

**UNIX®**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)/dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*/\*"  
...
```

### 后续步骤 {#next-steps-2}

您现在必须先执行以下操作，然后才能开始使用 Dispatcher：

* [配置](dispatcher-configuration.md) Dispatcher
* [配置 AEM](page-invalidate.md) 以使用 Dispatcher。
