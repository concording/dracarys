# 各种安全证书间的关系及相关操作


[舌尖上的大胖](https://www.jianshu.com/u/9a09120967a0) 关注


> 本文部分摘录于 [SSL Shopper](https://link.jianshu.com/?t=http%3A%2F%2Fwww.sslshopper.com)，这里提供了大量有价值的信息。

#### 一、证书标准

**1、X.509**
　　这是一种证书标准，主要定义了证书中应该包含哪些内容。其详情可以参考 `RFC5280`，SSL 使用的就是这种证书标准。

同样的 [X.509](https://link.jianshu.com/?t=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2FX.509) 证书，可能有不同的编码格式。最常见的就是 PEM 和 DER 两种格式。但是有个比较误导人的地方，这两种格式的文件，扩展名不一定就是 `.pem` 或 `.der`，还有其他一些常见格式。它们除了编码格式可能不同之外，内容也有差别。但是大多数都能相互转换编码格式。

**2、作为文件形式存在的证书一般有这几种格式：**

1.  带有私钥的证书（P12）
    　　由 Public Key Cryptography Standards #12，PKCS#12 标准定义，包含了**公钥**和**私钥**的**二进制格式**的证书形式，以 pfx 作为证书文件后缀名。
2.  二进制编码的证书（DER）
    　　证书中没有私钥，DER 编码二进制格式的证书文件，以 `.cer` 作为证书文件后缀名。
3.  Base64 编码的证书（PEM）
    　　证书中没有私钥，Base64 编码格式的证书文件，也是以 `.cer` 作为证书文件后缀名。

　　**由定义可以看出，只有 pfx/p12 格式的数字证书是包含有私钥的，cer 格式的数字证书里面只有公钥没有私钥。**

> 不同的平台和设备，需要不同格式的 SSL 证书。例如：

*   `.pfx` - Windows 平台使用
*   `.crt, .cer` - Apache Server 使用

> **注意：**区分 `PEM .pem` 和 `DER .cer` 唯一的办法，就是使用文本编辑器打开，查看 `BEGIN/END` 声明。

#### 二、主要格式

##### 1、PEM 格式

PEM（Privacy Enhanced Mail），OpenSSL 使用 PEM 格式来存放各种信息，它是 OpenSSL 默认采用的信息存放方式，是 CA（Certificate Authorities）颁发证书最常用的格式。包含 `—–BEGIN CERTIFICATE—–` 和 `—–END CERTIFICATE--` 声明。
　　多个 PEM 证书甚至于 Private Key 可以被包含到一个文件中，一个挨一个往下排布。但是大多数平台（如：Apache），希望证书和私钥分别存放到不同的文件中。

**文件一般包含如下信息：**

*   内容类型：
    　　表明本文件存放的是什么信息内容,它的形式为“——-BEGIN XXXX ——”,与结尾的“——END XXXX——”对应。

*   头信息：
    　　表明数据是如果被处理后存放，OpenSSL 中用的最多的是加密信息,比如加密算法以及初始化向量 iv。

*   信息体：
    　　为 BASE64 编码的数据。可以包括所有私钥（RSA 和 DSA）、公钥（RSA 和 DSA）和 (x509) 证书。它存储用 Base64 编码的 DER 格式数据，用 ascii 报头包围，因此适合系统之间的文本模式传输。

**PEM 格式文件特点：**

*   Base64 编码的 ASCII 文件
*   具备诸如 `.pem, .crt, .cer, .key` 这样的扩展名
*   Apache 和类似的服务器使用 PEM 格式的证书

查看 PEM 格式证书的信息：

```
$ openssl x509 -in certificate.pem -text -noout

```

使用 PEM 格式 存储的**_证书_**：

```
--BEGIN CERTIFICATE--
MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
......
1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
--END CERTIFICATE--

```

使用 PEM 格式存储的**_私钥_**：

```
--BEGIN RSA PRIVATE KEY--
MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
………
1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
--END RSA PRIVATE KEY--

```

使用 PEM 格式存储的证书**_请求文件_**：

```
--BEGIN CERTIFICATE REQUEST--
MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
………
1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
--END CERTIFICATE REQUEST--

```

##### 2、DER 格式

辨别编码规则 DER（Distinguished Encoding Rules）是 ASCII PEM 格式证书的二进制形式。所有类型的证书和私钥都可以被编码为 DER 格式。
　　它是大多数浏览器的缺省格式，并按 ASN1 DER 格式存储。它是无报头的。
　　PEM 是用文本报头包围的 DER。

*   二进制文件
*   扩展名为 `.cer` 和 `.der`
*   DER 被典型地于 Java 平台

查看 DER 格式证书的信息：

```
$ openssl x509 -in certificate.der -inform der -text -noout

```

##### 3、P7B/PKCS#7

PKCS7 – 加密消息语法（PKCS7），是各种消息存放的格式标准。这些消息包括：数据、签名数据、数字信封、签名数字信封、摘要数据和加密数据。
包含 `--BEGIN PKCS--` 和 `--END PKCS7--` 声明。可以包含证书和证书链，但是**_不包含私钥_**。

*   Base64 编码的 ASCII 文件
*   扩展名为 `.p7b, .p7c`
*   多平台支持。如：Windows OS、Java Tomcat

##### 4、PFX/PKCS#12

Predecessor of PKCS#12。PKCS12（个人数字证书标准）用于存放用户证书、crl、用户私钥以及证书链。PKCS12 中的私钥是加密存放的。
　　对 Unix 服务器来说，一般 `.crt` 和 `.key` 是分开存放在不同文件中的。但 Windows 的 IIS 则将它们存在一个 `.pfx` 文件中。因此这个文件包含了证书及私钥，这样会不会不安全？应该不会， `.pfx` 通常会有一个“提取密码”，你想把里面的东西读取出来的话，它就要求你提供提取密码。`.pfx` 使用的是 DER 编码。

**生成 `.pfx` 的方法：**

```
$ openssl pkcs12 -export -out certificate.pfx -inkey privateKey.key -in certificate.crt -certfile CACert.crt

```

其中 CACert.crt 是 CA 的根证书，有的话也通过 `-certfile` 参数一起带进去。

**PFX/PKCS#12 作用：**主要用于存储服务器证书，任何中间证书和私钥存于一个可加密文件中。

**PFX/PKCS#12 特点：**

*   二进制文件
*   扩展名为 `.pfx, .p12`
*   典型用于 Windows OS 导入导出证书和私钥

##### 5、CSR

Certificate Signing Request，即证书签名请求。这个并不是证书，而是向 CA 获得签名证书的申请。其核心内容是一个公钥（当然还附带了一些别的信息），在生成这个申请的时候，同时也会生成一个私钥，私钥要自己保管好。做过 iOS App 的朋友都应该知道是怎么向苹果申请开发者证书。
`.csr` 查看方法：

```
# 如果是 DER 格式的话照旧加上 -inform der
$ openssl req -noout -text -in my.csr

```

生成 X509 数字证书前,一般先由用户提交证书申请文件,然后由 CA 来签发证书。大致过程如下:

1.  用户生成自己的公私钥对
    　　构造自己的证书申请文件，符合 PKCS#10 标准。该文件主要包括了用户信息、公钥以及一些可选的属性信息，并用自己的私钥给该内容签名;

2.  用户将证书申请文件提交给 CA
    　　CA 验证签名，提取用户信息，并加上其他信息（比如：颁发者等信息），用 CA 的私钥签发数字证书;

**说明：**数字证书（如x.509）是将用户（或其他实体）身份与公钥绑定的信息载体。一个合法的数字证书不仅要符合 X509 格式规范，还必须有 CA 的签名。用户不仅有自己的数字证书，还必须有对应的私钥。X509 v3 数字证书主要包含的内容有：证书版本、证书序列号、签名算法、颁发者信息、有效时间、持有者信息、公钥信息、颁发者 ID、持有者 ID 和扩展项。

**操作举例**

向 CA 申请证书：

```
$ openssl req -newkey rsa:2048 -new -nodes -keyout my.key -out my.csr

```

把 `.csr` 交给 CA，CA 对此进行签名，完成。保留好 `.csr`，当权威证书颁发机构颁发的证书过期的时候，还可以用同样的 `.csr` 来申请新的证书，key 保持不变。

或者生成自签名的证书：

```
$ openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem

```

在生成证书的过程中会要你填一堆的东西，其实**真正要填的只有 Common Name**。通常填写你服务器的域名，如：`yourcompany.com`，或者你服务器的 IP 地址，其它都可以留空。

#### 三、其他扩展名的证书文件

##### `.crt`

应该是 Certification 的缩写。常见于 Unix 系统，编码有可能是 PEM，也可能是 DER，大多数应该是 PEM 编码。

##### `.cer`

应该是 Certification 的缩写。常见于 Windows 系统，编码有可能是 PEM，也可能是 DER，大多数应该是 DER 编码。

##### `.key`

通常用来存放一个公钥或者私钥，并非 X.509 证书，编码同样可能是 PEM 或 DER。

#### 四、证书不同格式间的转换

##### 1、PEM

**PEM 转为 DER**

```
$ openssl x509 -outform der -in certificate.pem -out certificate.der

```

**PEM 转为 P7B**

```
$ openssl crl2pkcs7 -nocrl -certfile certificate.cer -out certificate.p7b -certfile CAcert.cer

```

**PEM 转为 PFX**

```
$ openssl pkcs12 -export -out certificate.pfx -inkey privateKey.key -in certificate.crt -certfile CAcert.crt

```

##### 2、DER

**DER 转为 PEM**

```
$ openssl x509 -inform der -in certificate.cer -out certificate.pem

```

##### 3、P7B

**P7B 转换为 PEM**

```
$ openssl pkcs7 -print_certs -in certificate.p7b -out certificate.cer

```

**P7B 转换为 PFX**

```
# 先将 P7B 转为 PEM
$ openssl pkcs7 -print_certs -in certificate.p7b -out certificate.cer

# 再将 PEM 转为 PFX/P12
$ openssl pkcs12 -export -in certificate.cer -inkey privateKey.key -out certificate.pfx -certfile CAcert.cer

```

##### 4、PFX

**`x509` 到 `pfx`：**

```
$ openssl pkcs12 -export -in client.crt -inkey client.key -out client.pfx

```

**`PKCS#12` 与 `PEM` 的相互转换：**

```
# 从 cert.p12 中提取 私钥，不包含证书，私钥不加密，输出为 PEM 格式
$ openssl pkcs12 -nocerts -nodes -in cert.p12 -out privatekey.pem

# 从 cert.p12 中，只输出 client 证书，不输出私钥，输出为 PEM 格式
$ openssl pkcs12 -clcerts -nokeys -in cert.p12 -out cert.pem

# 将密钥与证书合成为 cert.p12
$ openssl pkcs12 -export -in cert.pem -out cert.p12 -inkey key.pem

```

从 `PFX` 格式文件中提取**`私钥`**格式文件（`.key`）：

```
$ openssl pkcs12 -in mycert.pfx -nocerts -nodes -out mycert.key

```

**PFX 转换为 PEM**

```
$ openssl pkcs12 -in certificate.pfx -out certificate.cer -nodes

```

**注意：**将 PFX 转换为 PEM 格式时，OpenSSL 会将所有的证书和私钥保存到一个文件中。需要使用文本编辑器打开，将每个证书和私钥（包括 `BEGIN/END` 声明）保存到自己独立的文本文件中，并且分别保存为 `certificate.cer, CAcert.cer, privateKey.key`。

##### 5、BKS

在一些 Java 环境（如 Android）中，需要 BKS 格式证书。一般做法是将 P12 转为 BKS：

```
# 先使用 openssl 把 crt 和 key 转换为 p12 证书
$ openssl pkcs12 -export -in client.crt -inkey client.key -out client.p12

# 使用 keytool 把 p12 转换为 bks 证书
$ keytool -importkeystore -srckeystore client.p12 -srcstoretype pkcs12 -destkeystore client.bks -deststoretype bks -provider org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath bcprov-ext-jdk15on-158.jar

```

使用 KeyTool 转换为 BKS 格式时，需要 `bcprov-ext-jdk15on-158.jar`，可以在[这里](https://link.jianshu.com/?t=https%3A%2F%2Fpan.baidu.com%2Fs%2F1lJzdYykh1c2DT_MPM0j9EQ)找到。文件路径直接带在 `-providerpath` 参数后面即可。也可以放到如下路径：`jdk/jre/lib/ext`。

* * *

#### 五、一些其他格式的介绍

##### JKS

Java Key Storage，这是 Java 的专利，跟 OpenSSL 关系不大。利用 Java 的一个叫 keytool 的工具，可以将 `.pfx` 转为 `.jks`，keytool 也能直接生成 `.jks`。JKS 文件格式被广泛的应用在基于 Java 的 Web 服务器、应用服务器、中间件。可以将 JKS 文件导入到Tomcat、 WebLogic 等软件。

##### KDB

通常可以将 Apache/OpenSSL 使用的“KEY文件 + CRT文件”格式转换为标准的 IBM KDB文件。KDB 文件格式被广泛的应用在 IBM 的 WEB 服务器、应用服务器、中间件。你可以将 KDB 文件导入到 IBM HTTP Server、IBM Websphere 等软件。

##### OCSP

在线证书状态协议（Online Certificate StatusProtocol, rfc2560），于实时表明证书状态。OCSP 客户端通过查询 OCSP 服务来确定一个证书的状态，可以提供给使用者一个或多个数字证书的有效性资料，它建立一个可实时响应的机制，让用户可以实时确认每一张证书的有效性，解决由CRL引发的安全问题。OCSP 可以通过 HTTP 协议来实现。rfc2560 定义了 OCSP 客户端和服务端的消息格式。

##### CRL

证书吊销列表（Certification Revocation List）是一种包含撤销的证书列表的签名数据结构。CRL 是证书撤销状态的公布形式，CRL 就像信用卡的黑名单，用于公布某些数字证书不再有效。CRL是一种离线的证书状态信息。它以一定的周期进行更新。CRL 可以分为完全 CRL和增量 CRL。在完全 CRL中包含了所有的被撤销证书信息，增量 CRL 由一系列的 CRL 来表明被撤销的证书信息，它每次发布的 CRL 是对前面发布 CRL的增量扩充。基本的 CRL 信息有:被撤销证书序列号、撤销时间、撤销原因、签名者以及 CRL 签名等信息。基于 CRL 的验证是一种不严格的证书认证。CRL 能证明在 CRL 中被撤销的证书是无效的。但是，它不能给出不在 CRL中的证书的状态。如果执行严格的认证,需要采用在线方式进行认证,即 OCSP认证。一般是由CA签名的一组电子文档，包括了被废除证书的唯一标识（证书序列号），CRL用来列出已经过期或废除的数字证书。它每隔一段时间就会更新，因此必须定期下载该清单，才会取得最新信息。

##### SCEP

简单证书注册协议。基于文件的证书登记方式需要从您的本地计算机将文本文件复制和粘贴到证书发布中心，和从证书发布中心复制和粘贴到您的本地计算机。 SCEP可以自动处理这个过程但是CRLs仍然需要手工的在本地计算机和CA发布中心之间进行复制和粘贴。

#### 六、参考资料

* * *

1.  原文：[What are the differences between PEM, DER, P7B/PKCS#7, PFX/PKCS#12 certificates](https://link.jianshu.com/?t=https%3A%2F%2Fmyonlineusb.wordpress.com%2F2011%2F06%2F19%2Fwhat-are-the-differences-between-pem-der-p7bpkcs7-pfxpkcs12-certificates%2F)
2.  参考 [《那些证书相关的玩意儿（SSL,X.509,PEM,DER,CRT,CER,KEY,CSR,P12等）》](https://link.jianshu.com/?t=http%3A%2F%2Fwww.360doc.com%2Fcontent%2F15%2F0520%2F10%2F21412_471902987.shtml) 部分内容做了补充
3.  [常见数字证书及协议介绍](https://www.jianshu.com/p/f32852523f1b)
4.  [RSA私钥和公钥文件格式 (pkcs#7, pkcs#8, pkcs#12, pem)](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2Ftuhuolong%2Farticle%2Fdetails%2F42778945)
5.  [ASN.1 key structures in DER and PEM](https://link.jianshu.com/?t=https%3A%2F%2Ftls.mbed.org%2Fkb%2Fcryptography%2Fasn1-key-structures-in-der-and-pem)
6.  [openSSL将.crt证书生成.bks](https://link.jianshu.com/?t=https%3A%2F%2Fblog.csdn.net%2Fqq_36992688%2Farticle%2Fdetails%2F78861883)