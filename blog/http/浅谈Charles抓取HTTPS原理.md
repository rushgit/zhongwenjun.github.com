在[关于HTTPS，你需要知道的全部](./关于HTTPS，你需要知道的全部.md)中，分析了HTTPS的安全通信过程，知道了HTTPS可以有效防止中间人攻击。但用过抓包工具的人都知道，比如Charles，Fiddler是可以抓取HTTPS请求并解密的，它们是如何做到的呢？

首先来看Charles官网对HTTPS代理的描述：

Charles can be used as a man-in-the-middle HTTPS proxy, enabling you to view in plain text the communication between web browser and SSL web server.`

`Charles does this by becoming a man-in-the-middle. Instead of your browser seeing the server’s certificate, Charles dynamically generates a certificate for the server and signs it with its own root certificate (the Charles CA Certificate). Charles receives the server’s certificate, while your browser receives Charles’s certificate. Therefore you will see a security warning, indicating that the root authority is not trusted. If you add the Charles CA Certificate to your trusted certificates you will no longer see any warnings – see below for how to do this.
`

Charles作为一个[“中间人代理”](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB)，当浏览器和服务器通信时，Charles接收服务器的证书，但动态生成一张证书发送给浏览器，也就是说Charles作为中间代理在浏览器和服务器之间通信，所以通信的数据可以被Charles拦截并解密。由于Charles更改了证书，浏览器校验不通过会给出安全警告，必须安装Charles的证书后才能进行正常访问。

下面来看具体的流程：

![Charles抓取HTTPS流程](http://upload-images.jianshu.io/upload_images/2438937-a453e3e411efcae5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 客户端向服务器发起HTTPS请求

2. Charles拦截客户端的请求，伪装成客户端向服务器进行请求

3. 服务器向“客户端”（实际上是Charles）返回服务器的CA证书

4. Charles拦截服务器的响应，获取服务器证书公钥，然后自己制作一张证书，将服务器证书替换后发送给客户端。**（这一步，Charles拿到了服务器证书的公钥）**

5. 客户端接收到“服务器”（实际上是Charles）的证书后，生成一个对称密钥，用Charles的公钥加密，发送给“服务器”（Charles）

6. Charles拦截客户端的响应，用自己的私钥解密对称密钥，然后用服务器证书公钥加密，发送给服务器。**（这一步，Charles拿到了对称密钥）**

7. 服务器用自己的私钥解密对称密钥，向“客户端”（Charles）发送响应

8. Charles拦截服务器的响应，替换成自己的证书后发送给客户端

9. 至此，连接建立，Charles拿到了 服务器证书的公钥 和 客户端与服务器协商的对称密钥，之后就可以解密或者修改加密的报文了。

HTTPS抓包的原理还是挺简单的，简单来说，就是Charles作为**“中间人代理”**，拿到了 **服务器证书公钥** 和 **HTTPS连接的对称密钥**，前提是客户端选择信任并安装Charles的CA证书，否则客户端就会“报警”并中止连接。这样看来，HTTPS还是很安全的。