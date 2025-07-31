---
title: 将 SSL 与 Dispatcher 结合使用
description: 了解如何配置 Dispatcher 以通过 SSL 连接与 AEM 通信。
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
index: y
internal: n
snippet: y
exl-id: ec378409-ddb7-4917-981d-dbf2198aca98
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: ht
source-wordcount: '1305'
ht-degree: 100%

---

# 在 Dispatcher 中使用 SSL {#using-ssl-with-dispatcher}

在 Dispatcher 和渲染计算机之间使用 SSL 连接：

* [单向 SSL](#use-ssl-when-dispatcher-connects-to-aem)
* [双向 SSL](#configuring-mutual-ssl-between-dispatcher-and-aem)

>[!NOTE]
>
>与 SSL 证书相关的操作绑定到第三方产品。它们不包含在 Adobe 白金维护与支持合同的保障范围内。

## Dispatcher 与 AEM 连接时使用 SSL {#use-ssl-when-dispatcher-connects-to-aem}

配置 Dispatcher，使其通过 SSL 连接与 AEM 或 CQ 渲染实例进行通信。

在配置 Dispatcher 之前，请先配置 AEM 或 CQ 以使用 SSL。有关更多信息，请参阅：

* [默认启用 SSL/TLS](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/security/ssl-by-default)
* [在 AEM 中使用 SSL 向导](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-learn/foundation/security/use-the-ssl-wizard)

### 与 SSL 相关的请求标头 {#ssl-related-request-headers}

当 Dispatcher 收到 HTTPS 请求时，Dispatcher 将以下标头包括在发送到 AEM 或 CQ 的后续请求中：

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

通过 Apache 2.4 使用 `mod_ssl` 发起的请求将包含类似以下示例的标头：

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### 将 Dispatcher 配置为使用 SSL {#configuring-dispatcher-to-use-ssl}

要将 Dispatcher 配置为使用 SSL 与 AEM 或 CQ 连接，您的 [dispatcher.any](dispatcher-configuration.md) 文件需要以下属性：

* 一个处理 HTTPS 请求的虚拟主机。
* 虚拟主机的 `renders` 部分包含一项，该项标识使用 HTTPS 的 CQ 或 AEM 实例的主机名和端口。
* `renders` 项包含一个名为 `secure` 的属性，其值为 `1`。

注意：如有必要，请创建另一个虚拟主机以处理 HTTP 请求。

以下示例 `dispatcher.any` 文件展示用于通过 HTTPS 连接到在主机 `localhost` 和端口 `8443` 上运行的 CQ 实例的属性值：

```
/farms
{
   /secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTPS requests
         "https://*"
      }
      /renders
      {
      /0001
         {
            # hostname or IP of the render
            /hostname "localhost"
            # port of the render
            /port "8443"
            # connect via HTTPS
            /secure "1"
         }
      }
     # the rest of the properties are omitted
   }

   /non-secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTP requests
         "http://*"
      }
      /renders
      {
         /0001
      {
         # hostname or IP of the render
         /hostname "localhost"
         # port of the render
         /port "4503"
      }
   }
    # the rest of the properties are omitted
}
```

## 配置 Dispatcher 与 AEM 之间的双向 SSL {#configuring-mutual-ssl-between-dispatcher-and-aem}

要使用双向 SSL，请配置 Dispatcher 与渲染计算机（一般为 AEM 或 CQ 发布实例）之间的连接：

* Dispatcher 通过 SSL 连接到渲染实例。
* 渲染实例验证 Dispatcher 证书的有效性。
* Dispatcher 验证渲染实例的证书 CA 是否可信。
* （可选）Dispatcher 验证渲染实例的证书是否与渲染实例的服务器地址匹配。

要配置双向 SSL，您需要由受信任的证书颁发机构 (CA) 签名的证书。自签名证书无法满足需求。您可以充当 CA 或使用第三方 CA 的服务来签署证书。要配置双向 SSL，您需要以下项：

* 渲染实例和 Dispatcher 的签名证书
* CA 证书（如果您充当 CA）
* 用于生成 CA、证书和证书请求的 OpenSSL 库。

要配置双向 SSL，请执行以下步骤：

1. [安装](dispatcher-install.md)适用于您的平台的最新版本的 Dispatcher。使用支持 SSL 的 Dispatcher 二进制文件（SSL 在文件名中，例如 `dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar`）。
1. 为 Dispatcher 和渲染实例[创建或获取 CA 签名证书](dispatcher-ssl.md#main-pars-title-3)。
1. [创建包含渲染证书的密钥库](dispatcher-ssl.md#main-pars-title-6)并配置渲染器的 HTTP 服务。
1. 为双向 SSL [配置 Dispatcher Web Server 模块](dispatcher-ssl.md#main-pars-title-4)。

### 创建或获取 CA 签名证书 {#creating-or-obtaining-ca-signed-certificates}

创建或获取由 CA 签名的证书，用于验证发布实例和 Dispatcher 的身份。

#### 创建您的 CA {#creating-your-ca}

如果您充当 CA，请使用 [OpenSSL](https://www.openssl.org/) 创建用于签署服务器和客户端证书的证书颁发机构。（您必须安装 OpenSSL 库。）如果您使用第三方 CA，请不要执行此过程。

1. 打开终端并将当前目录更改为包含 `CA.sh` 文件的目录，例如 `/usr/local/ssl/misc`。
1. 要创建 CA，请输入以下命令，然后在出现提示时提供值：

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >`openssl.cnf` 文件中的若干属性控制 CA.sh 脚本的行为。在创建 CA 之前，根据需要编辑此文件。

#### 创建证书 {#creating-the-certificates}

使用 OpenSSL 可创建要发送给第三方 CA 或通过您的 CA 签名的证书请求。

创建证书时，OpenSSL 使用公用名属性来标识证书所有者。对于渲染实例的证书，如果您配置 Dispatcher 以接受证书，则请使用实例计算机的主机名作为通用名称。仅当该名称与发布实例的主机名匹配时才执行此程序。参阅 [DispatcherCheckPeerCN](dispatcher-ssl.md#main-pars-title-11) 属性。

1. 打开终端并将当前目录更改为包含 OpenSSL 库的 CH.sh 文件的目录。
1. 输入以下命令，并在系统提示时提供值。如有必要，可使用发布实例的主机名作为公用名。主机名是渲染器的 IP 地址的 DNS 可解析名称：

   ```shell
   ./CA.sh -newreq
   ```

   如果您使用的是第三方 CA，请将 newreq.pem 文件发送到 CA 以进行签名。如果您充当 CA，请继续执行步骤 3。

1. 要使用您 CA 的证书签署该证书，请输入以下命令：

   ```shell
   ./CA.sh -sign
   ```

   在包含 CA 管理文件的目录中创建两个分别名为 `newcert.pem` 和 `newkey.pem` 的文件。这两个文件分别是渲染计算机的公共证书和私钥。

1. 将 `newcert.pem` 重命名为 `rendercert.pem`，将 `newkey.pem` 重命名为 `renderkey.pem`。
1. 重复第 2 步和第 3 步以创建 Dispatcher 模块的证书和公钥。确保您使用的是特定于 Dispatcher 实例的公用名。
1. 将 `newcert.pem` 重命名为 `dispcert.pem`，将 `newkey.pem` 重命名为 `dispkey.pem`。

### 在渲染计算机上配置 SSL {#configuring-ssl-on-the-render-computer}

使用 `rendercert.pem` 和 `renderkey.pem` 文件在渲染实例上配置 SSL。

#### 将渲染证书转换为 JKS（Java™ 密钥库）格式 {#converting-the-render-certificate-to-jks-format}

使用以下命令将渲染证书（一个 PEM 文件）转换为 PKCS#12 文件。还包括签署呈现证书的 CA 的证书：

1. 在终端窗口中，将当前目录更改为渲染证书和私钥的位置。
1. 要将渲染证书（一个 PEM 文件）转换为 PKCS#12 文件，请输入以下命令。还包括为渲染证书签名的 CA 证书：

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. 要将 PKCS#12 文件转换为 Java™ KeyStore (JKS) 格式，请输入以下命令：

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. 使用默认别名创建 Java™ Keystore。需要时可更改别名：

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### 将 CA 证书添加到渲染器的信任库中 {#adding-the-ca-cert-to-the-render-s-truststore}

如果您充当 CA，请将您的 CA 证书导入密钥库中。然后，将运行渲染实例的 JVM 配置为信任密钥库。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. 使用文本编辑器打开 cacert.pem 文件，然后删除所有在以下各行之前的文本：

   `-----BEGIN CERTIFICATE-----`

1. 使用以下命令将证书导入密钥库中：

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. 要将运行渲染实例的 JVM 配置为信任密钥库，请使用以下系统属性：

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   例如，如果您使用 crx-quickstart/bin/quickstart 脚本来启动您的发布实例，则可修改 CQ_JVM_OPTS 属性：

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### 配置渲染实例 {#configuring-the-render-instance}

若要将渲染实例的 HTTP 服务配置为使用 SSL，请按照 *`Enable SSL on the Publish Instance`* 部分中的说明使用渲染证书：

* AEM 6.2：[启用 HTTP Over SSL](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)
* AEM 6.1：[启用 HTTP Over SSL](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)
* 较旧的 AEM 版本：请参阅[此页面。](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)

### 为 Dispatcher 模块配置 SSL {#configuring-ssl-for-the-dispatcher-module}

要将 Dispatcher 配置为使用双向 SSL，需先准备 Dispatcher 证书，然后配置 Web 服务器模块。

### 创建统一的 Dispatcher 证书 {#creating-a-unified-dispatcher-certificate}

将 Dispatcher 证书和未加密的私钥合并为单个 PEM 文件。使用文本编辑器或 `cat` 命令创建一个与以下示例类似的文件：

1. 打开终端并将当前目录更改为 dispkey.pem 文件的位置。
1. 要解密私钥，请输入以下命令：

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. 使用文本编辑器或 `cat` 命令将未加密的私钥和证书并入一个类似于以下示例的文件中：

   ```xml
   -----BEGIN RSA PRIVATE KEY-----
   MIICxjBABgkqhkiG9w0B...
   ...M2HWhDn5ywJsX
   -----END RSA PRIVATE KEY-----
   -----BEGIN CERTIFICATE-----
   MIIC3TCCAk...
   ...roZAs=
   -----END CERTIFICATE-----
   ```

### 指定 Dispatcher 所使用的证书 {#specifying-the-certificate-to-use-for-dispatcher}

将以下属性添加到 [Dispatcher 模块配置](dispatcher-install.md#main-pars-55-35-1022)（在 `httpd.conf` 中）：

* `DispatcherCertificateFile`：Dispatcher 统一证书文件的路径，包含公共证书和未加密的私钥。在 SSL 服务器请求 Dispatcher 客户端证书时使用此文件。
* `DispatcherCACertificateFile`：CA 证书文件的路径。当 SSL 服务器提供的 CA 证书不受根证书颁发机构信任时使用。
* `DispatcherCheckPeerCN`：为远程服务器证书启用 (`On`) 还是禁用 (`Off`) 主机名检查。

以下代码是示例配置：

```xml
<IfModule disp_apache2.c>
  DispatcherConfig conf/dispatcher.any
  DispatcherLog    logs/dispatcher.log
  DispatcherLogLevel 3
  DispatcherNoServerHeader 0
  DispatcherDeclineRoot 0
  DispatcherUseProcessedURL 0
  DispatcherPassError 0
  DispatcherCertificateFile disp_unified.pem
  DispatcherCACertificateFile cacert.pem
  DispatcherCheckPeerCN On
</IfModule>
```
