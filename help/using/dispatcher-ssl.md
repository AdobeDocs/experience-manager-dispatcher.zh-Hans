---
title: 将SSL与调度程序结合使用
seo-title: 将SSL与调度程序结合使用
description: 了解如何配置Dispatcher以使用SSL连接与AEM进行通信。
seo-description: 了解如何配置Dispatcher以使用SSL连接与AEM进行通信。
uuid: 1a8f448c-d3 d8-4798-a5 cb-9579171171eed
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/Dispatcher
topic-tags: 调度程序
content-type: 引用
discoiquuid: 771CFD85-6c26-4ff2-a3 fe-dff8 d8 f7920 b
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: 8dd56f8b90331f0da43852e25893bc6f3e606a97

---


# 将SSL与调度程序结合使用 {#using-ssl-with-dispatcher}

在Dispatcher和渲染计算机之间使用SSL连接：

* [单向SSL](dispatcher-ssl.md#main-pars-title-1)
* [相互SSL](dispatcher-ssl.md#main-pars-title-2)

>[!NOTE]
>
>与SSL证书相关的操作绑定到第三方产品。它们未被Adobe白金维护和支持合同覆盖。

## 调度程序连接到AEM时使用SSL {#use-ssl-when-dispatcher-connects-to-aem}

配置Dispatcher以使用SSL连接与AEM或CQ渲染实例进行通信。

配置Dispatcher之前，请配置AEM或CQ以使用SSL：

* AEM6.2： [启用HTTP over SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM6.1： [启用HTTP over SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 旧版AEM版本：查看 [此页面](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)。

### SSL相关请求标题 {#ssl-related-request-headers}

当调度程序接收HTTPS请求时，调度程序在发送到AEM或CQ的后续请求中包含以下标题：

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

通过Apache-2.2发出的请求 `mod_ssl` 包含类似于以下示例的标题：

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### 配置调度程序以使用SSL {#configuring-dispatcher-to-use-ssl}

要配置Dispatcher以通过SSL或CQ通过SSL连接 [到您](dispatcher-configuration.md) 的调度程序，请使用下列属性：

* 处理HTTPS请求的虚拟主机。
* 虚拟主机的 `renders` 部分包括一个项目，用于标识使用HTTPS的CQ或AEM实例的主机名和端口。
* `renders` 项目中包含一个名为 `secure` value `1`的属性。

注意：根据需要创建其他虚拟主机以处理HTTP请求。

以下示例调度程序。任何文件都显示使用HTTPS连接到主机 `localhost` 和端口上运行的CQ实例的属性值 `8443`：

```
/farms
{
   /secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTPS requestss
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
         "https://*"
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

## 在调度程序和AEM之间配置相互SSL {#configuring-mutual-ssl-between-dispatcher-and-aem}

配置Dispatcher与渲染计算机(通常是AEM或CQ发布实例)之间的连接以使用相互SSL：

* 调度程序通过SSL连接到渲染实例。
* 渲染实例验证Dispatcher证书的有效性。
* 调度程序验证渲染实例证书的CA是否可信。
* (可选)调度程序验证渲染实例的证书是否与渲染实例的服务器地址匹配。

要配置共同SSL，您需要由受托证书颁发机构(CA)签名的证书。自签名证书是不够的。您可以充当CA或使用第三方CA的服务签署证书。要配置共同SSL，您需要以下项目：

* 渲染实例和调度程序的签名证书
* CA证书(如果您充当CA)
* 用于生成CA、证书和证书请求的OpenSSL库。

执行以下步骤配置相互SSL：

1. [为](dispatcher-install.md) 平台安装Dispatcher的最新版本。使用支持SSL的Dispatcher二进制文件(SSL在文件名中，如dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar)。
1. [为Dispatcher和渲染实例创建或获取CA签名证书](dispatcher-ssl.md#main-pars-title-3) 。
1. [创建包含渲染证书的keystore](dispatcher-ssl.md#main-pars-title-6) 并配置渲染的HTTP服务以使用它。
1. [为共同SSL配置Dispatcher Web服务器模块](dispatcher-ssl.md#main-pars-title-4) 。

### 创建或获取CA签名的证书 {#creating-or-obtaining-ca-signed-certificates}

创建或获取验证发布实例和调度程序的CA签名证书。

#### 创建CA {#creating-your-ca}

如果您充当CA，请使用 [OpenSSL](https://www.openssl.org/) 创建签署服务器和客户端证书的证书颁发机构。(必须安装OpenSSL库。)如果您使用的是第三方CA，请勿执行此过程。

1. 打开一个终端并将当前目录更改为连接CA. sh文件的目录，如 `/usr/local/ssl/misc`。
1. 要创建CA，请输入以下命令，然后在promted时提供值：

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >opensl. cnf文件中的几个属性控制CA. sh脚本的行为。在创建CA之前，应根据需要修改此文件。

#### 创建证书 {#creating-the-certificates}

使用OpenSSL创建要发送给第三方CA或与CA签名的证书请求。

创建证书时，OpenSSL使用“通用名称”属性标识证书持有人。对于渲染实例的证书，如果配置Dispatcher以仅接受证书的主机名(如果它与Publish实例的主机名匹配)，则使用实例计算机的主机名作为通用名称。(请参阅 [dispatcherCheckPeerCN](dispatcher-ssl.md#main-pars-title-11) 属性。)

1. 打开一个终端并将当前目录更改为包含OPensSL库的CH. sh文件的目录。
1. 输入以下命令，并在出现提示时提供值。如果需要，请使用发布实例的主机名作为通用名称。主机名是DNS地址的DNS可解析名称：

   ```shell
   ./CA.sh -newreq
   ```

   如果您使用的是第三方CA，请将newreq. pem文件发送给CA以进行签名。如果您充当CA，请继续执行步骤3。

1. 输入以下命令以使用CA的证书对证书进行签名：

   ```shell
   ./CA.sh -sign
   ```

   在包含CA管理文件的目录中创建了两个名为newcert. pem和newkey. pem的文件。它们分别是渲染计算机的公共证书和私钥。

1. 将newcert. pem重命名为renercert. pem，并将newkey. pem重命名为enderkey. pem。
1. 重复步骤和步骤3，为Dispatcher模块创建新证书和新公钥。确保您使用的是特定于调度程序实例的通用名称。
1. 重命名newcert. pem以清除. pem，并将newkey. pem重命名为dispey. pem。

### 在渲染计算机上配置SSL {#configuring-ssl-on-the-render-computer}

使用渲染器. pem和renderkey. pem文件在渲染实例上配置SSL。

#### 将渲染证书转换为JKS格式 {#converting-the-render-certificate-to-jks-format}

使用以下逗号将渲染证书(即PEM文件)转换为PKCS#12文件。还包括签署渲染证书的CA证书：

1. 在终端窗口中，将当前目录更改为渲染证书和私钥的位置。
1. 输入以下逗号，将渲染证书(即PEM文件)转换为PKCS#12文件。还包括签署渲染证书的CA证书：

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. 输入以下命令将PKCS#12文件转换为Java Keystore(JKS)格式：

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Java Keystore是使用默认别名创建的。根据需要更改别名：

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### 将CA Cert添加到渲染的信任商店 {#adding-the-ca-cert-to-the-render-s-truststore}

如果您充当CA，请将CA证书导入keystore。然后，配置运行渲染实例以信任keystore的JVM。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. 使用文本编辑器打开cacert. pem文件并删除以下下一行的所有文本：

   `-----BEGIN CERTIFICATE-----`

1. 使用以下命令将证书导入keystore：

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. 要配置运行渲染实例以信任keystore的JVM，请使用以下系统属性：

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   例如，如果使用crx-quickstart/bin/quickstart脚本启动发布实例，则可以修改CQ_ JVM_ OPTS属性：

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### 配置渲染实例 {#configuring-the-render-instance}

使用渲染证书在Publish实例 ** 部分启用SSL中的说明，将渲染实例的HTTP服务配置为使用SSL：

* AEM6.2： [启用HTTP over SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM6.1： [启用HTTP over SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 旧版AEM版本：查看 [此页面。](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)

### 为调度程序模块配置SSL {#configuring-ssl-for-the-dispatcher-module}

要配置Dispatcher以使用共同SSL，请准备Dispatcher证书，然后配置Web服务器模块。

### 创建统一的调度程序证书 {#creating-a-unified-dispatcher-certificate}

将调度程序证书和未加密的私钥合并到一个PEM文件中。使用文本编辑器或 `cat` 命令创建类似于以下示例的文件：

1. 打开一个终端，并将当前目录更改为dispey. pem文件的位置。
1. 要解密私钥，请输入以下命令：

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. 使用文本编辑器或 `cat` 命令将未加密的私钥和证书合并到一个文件中，该文件类似于以下示例：

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

### 指定要用于调度程序的证书 {#specifying-the-certificate-to-use-for-dispatcher}

将以下属性添加到 [Dispatcher模块配置](dispatcher-install.md#main-pars-55-35-1022) (英寸 `httpd.conf`)：

* `DispatcherCertificateFile`：调度程序统一证书文件的路径，该文件包含公用证书和未加密的私钥。SSL服务器请求Dispatcher客户端证书时使用此文件。
* `DispatcherCACertificateFile`：CA证书文件的路径，如果SSL服务器呈现不受根授权信任的CA。
* `DispatcherCheckPeerCN`：是否为远程服务器证书启用( `On`)或disable( `Off`)主机名检查。

以下代码是一个示例配置：

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
