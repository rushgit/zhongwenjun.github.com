# APK签名机制原理详解

<br>

# 前言

众所周知，Android系统在安装Apk的过程中，会对Apk进行签名校验，校验通过后才能安装成功。那你知道签名校验的机制是什么？具体校验的是什么内容吗？申请第三方SDK（如微信支付）时填入的SAH1值是什么？目前众多的快速批量打包方案又是如何绕过签名检验的？

我将通过一系列的文章来解开这些疑惑：

1. [Apk签名的基本概念和用法（本篇）](sign_mechanism.md)
2. [Apk签名机制之——JAR签名机制详解](./jar_sign.md)
3. [Apk签名机制之——V2签名机制详解](./apk_sign.md)

这篇文章先来介绍Apk签名相关的基本知识。

1. 签名是什么？如何进行签名？
2. keystore和证书格式
3. jarsigner和apksigner的区别

# 1. 签名是什么？

要知道签名是什么，先来看为什么需要签名 。大家都知道，在消息通信时，必须至少解决两个问题：一是确保消息来源的真实性，二是确保消息不会被第三方篡改。在安装Apk时，同样需要确保Apk来源的真实性，以及Apk没有被第三方篡改。如何解决这两个问题呢？方法就是开发者对Apk进行签名：在Apk中写入一个“指纹”。指纹写入以后，Apk中有任何修改，都会导致这个指纹无效，Android系统在安装Apk进行签名校验时就会不通过，从而保证了安全性。

要了解如何实现签名，需要了解两个基本概念：数字摘要和数字证书。

## 1.1 数字摘要

> **数字摘要**是将任意长度的消息变成固定长度的短消息，它类似于一个自变量是消息的函数，也就是Hash函数。数字摘要就是采用单向Hash函数将需要加密的明文“摘要”成一串固定长度的密文，这一串密文又称为数字指纹，它有固定的长度，而且不同的明文摘要成密文，其结果总是不同的，而同样的明文其摘要必定一致。

简单来说，就是对一个任意长度的数据，通过一个Hash算法计算后，都可以得到一个固定长度的二进制数据，这个数据就称为“摘要”。摘要具有下面的几个特征：

1. 唯一性

   在不考虑碰撞的情况下，不同的数据的计算出的摘要是不同的。

2. 固定长度

   不同的Hash算法计算的长度是不一样的，但对同一个算法来说是一样的。比较常用的Hash算法有MD5和SHA1，MD5的长度是128拉，SHA1的长度是160位。

3. 不可逆性

   即从正向计算的摘要不可能逆向推导出原始数据。

## 1.2 签名和校验的大体过程

前面已经说到，可以通过签名来确保数据来源的可靠性和数据的不可篡改性。签名就是在摘要的基础上再进行一次加密，对摘要加密后的数据就可以当作数字签名，在安装Apk需要对签名进行验证，验证通过才能继续安装。

这里有两个过程：签名过程 和 校验过程。

![sign](./res/sign_verify.png)

先来说**签名过程：**

1. 计算摘要

   通过Hash算法提取出原始数据的摘要；

2. 计算签名

   再通过基于密钥（私钥）的非对称加密算法对提取出的摘要进行加密，加密后的数据就是签名信息；

3. 写入签名

   将签名信息写入原始数据的签名区块内。

再来看**校验过程：**

1. 计算摘要

   接收方接收到数据后，首先用同样的Hash算法从接收到的数据中提取出摘要；

2. 解密签名

   使用发送方的公钥对数字签名进行解密，解密出原始摘要；

3. 比较摘要

   如果解密后的数据和提取的摘要一致，则校验通过；如果数据被第三方篡改过，解密后的数据和摘要不一致，校验不通过。

## 1.3 数字证书

这里有一个前提：接收方必须要知道发送方的公钥和所使用的算法。如果数字签名和公钥一起被篡改，接收方无法得知，还是会校验通过。如何保证公钥的可靠性呢？答案是数字证书，数字证书是身份认证机构（Certificate Authority）颁发的，包含了以下信息：

- 证书颁发机构
- 证书颁发机构签名
- 证书绑定的服务器域名
- 证书版本、有效期
- 签名使用的加密算法（非对称算法，如RSA）
- 公钥 等

接收方收到消息后，先向CA验证证书的合法性（根据证书的签名、绑定的域名等信息。CA机构是权威的，可以保证这个过程的可靠性。）再进行签名校验。

> 关于通过数字证书进行签名校验的详细过程，可以参考我之前写的一篇关于HTTPS通信机制的介绍：[详谈HTTPS通信机制，HTTPS是如何进行安全通信的?](https://blog.csdn.net/zwjemperor/article/details/80719427#t9)

需要注意的是，Apk的证书通常的自签名的，也就是由开发者自己制作，没有向CA机构申请。Android在安装Apk时并没有校验证书本身的合法性，只是从证书中提取公钥和加密算法，这也正是对第三方Apk重新签名后，还能够继续在没有安装这个Apk的系统中继续安装的原因。

## 1.4 签名和校验过程 

完整的签名和校验过程如下：（图片来源：[维基百科](https://upload.wikimedia.org/wikipedia/commons/2/2b/Digital_Signature_diagram.svg)）

![2](./res/sign_verify_wiki.png)

# 2. keystore和证书格式

我们在对Apk签名时并没有直接指定私钥、公钥和数字证书，而是使用keystore文件，这些信息都包含在了keystore文件中。根据编码不同，keystore文件分为很多种，Android使用的是Java标准keystore格式JKS(Java Key Storage)，所以通过Android Studio导出的keystore文件是以.jks结尾的。

keystore使用的证书标准是X.509，X.509标准也有多种编码格式，常用的有两种：pem（Privacy Enhanced Mail）和der（Distinguished Encoding Rules）。jks使用的是der格式，Android也支持直接使用pem格式的证书进行签名，我们下面会介绍。

两种证书编码格式的区别：

* DER（Distinguished Encoding Rules）

  二进制格式，所有类型的证书和私钥都可以存储为der格式。

* PEM（Privacy Enhanced Mail）

  base64编码，内容以-----BEGIN xxx----- 开头，以-----END xxx----- 结尾，比如：

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAlmXFRXEZomRKhNRp2XRoXH+2hm17RfrfecQlT49fktoDLkF6r99uiNnuUdPi6UQuXOnzEbe1nZkfuqfB10aBLrDqBUSZ+3
-----END RSA PRIVATE KEY-----

-----BEGIN CERTIFICATE-----
MIICvTCCAaWgAwIBAgIEcWTElDANBgkqhkiG9w0BAQsFADAPMQ0wCwYDVQQDEwRyPQDLnVKeEIh81OwD3KIrQOUwsxyptOVVea1D8CzIAnGs
-----END CERTIFICATE-----
```

 <br>

**X.509证书格式：**

![cert](./res/cert.png)

#3. jarsigner和apksigner的区别

Android提供了两种对Apk的签名方式，一种是基于JAR的签名方式，另一种是基于Apk的签名方式，它们的主要区别在于使用的签名文件不一样：jarsigner使用keystore文件进行签名；apksigner除了支持使用keystore文件进行签名外，还支持直接指定pem证书文件和私钥进行签名。

~~~groovy
android {
    signingConfigs {
        config {
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword'] 
        }
    }
    ...
  }
~~~

不知道大家有没有注意一个问题，我们通过keytool或者AS生成一个keystore的时候（[签署您的应用](https://developer.android.com/studio/publish/app-signing)），除了要输入keystore的密码外，还要输入一个alias和key的密码。在签名时，除了要指定keystore文件和密码外，也要指定alias和key的密码，这是为什么呢？

原因是keystore是一个密钥库，也就是说它可以存储多对密钥和证书，keystore的密码是用于保护keystore本身的，一对密钥和证书是通过alias来区分的。从这里可以看出jarsigner是支持使用多个证书对Apk进行签名的。apksigner也同样支持，关于apksigner的使用介绍可以参考官方文档[apksigner](https://developer.android.com/studio/command-line/apksigner)。

## 3.1 签名相关命令

```shell
● jarsigner签名
jarsigner -keystore keystore_file -signedjar signed.apk unsigned.apk alias_name -storepass pwd

● apksigner签名
java -jar signapk.jar cert.x509.pem private.pk8 unsigned.apk signed.apk

● 查看keystore文件
keytool -list  -v -keystore keystore_file -storepass pwd

● 查看apk证书
keytool -printcert -jarfile apk

● 查看DER格式证书(META-INFO/CERT.RSA)
openssl pkcs7 -inform DER -in CERT.RSA -noout -print_certs -text

● 查看PEM格式证书
openssl x509 -in cert.x509.pem -text -noout

● apksigner检查apk是否签名，以及查看证书SHA1值
apksigner verify -v --print-certs

```


ok，签名的基本概念和校验过程就介绍到这里，关于JAR签名和V2签名机制的详细介绍，参考下面两篇文章：

* [Apk签名机制之——JAR签名机制详解](./jar_sign.md)
* [Apk签名机制之——V2签名机制详解](./apk_sign.md)