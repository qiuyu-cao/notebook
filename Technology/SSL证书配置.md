# SSL/TLS 证书

## 1 SSL/TLS 之间的关系

SSL (Secure Sockets Layer) 和 TLS (Secure Sockets Layer) 都是通讯加密协议。早起的 SSL 协议历史上存在过三个版本 (SSL 1.0、SSL 2.0 和 SSL 3.0) ，但是由于存在安全问题而逐渐被淘汰。TLS 是 SSL 的后继者，现在存在 TLS 1.0、1.1、1.2 和 1.3 等版本。虽然现在最常使用的是 TLS 协议，但是在习惯上仍然称作 SSL ，下文也按照习惯统称为 SSL。

## 2 加密实现原理

SSL 协议是为了让 http 通讯更加安全，支持 SSL 协议的 http 通讯通常会在浏览器地址栏输入 https。由于现在不支持 SSL 协议的网站是比较少见的，只支持 http 通讯的网站会被浏览器标记为 '不安全'。

SSL 协议的运行需要第三方 CA 机构作为客户端与服务端进行通讯的公证人。在服务端保存了两把钥匙：一把钥匙是公开的称为公钥，公钥可以被任何人获取。另一把只有自己保存，称为私钥。用公钥加密的信息只有使用私钥才能进行解密。CA 机构负责认证公钥和网站信息并进行公示。

假设在网站的服务端已经配置了 SSL 证书，通过在浏览器输入 https:// + 域名 进行访问，这个访问的过程是这样的：

1. 服务器返回证书
   当浏览器向服务器发送请求后，服务器返回一个包含以下信息的 TLS/SSL 证书：
   1. 主题信息：包括域名、组织名称和组织位置。
   2. 颁发机构 (CA) 信息。
   3. 证书的有效期。
   4. 证书序列号。
   5. 公钥。
   6. 数字签名：包含签名算法和签名。
   7. 扩展信息：可能包括密钥用途、备用名称等。
   8. （如果适用）域名信息：对于某些特殊类型的证书，如 EV SSL 证书。
2. 浏览器验证证书
   1. 浏览器使用内置的受信任证书列表来验证证书的信任链和数字签名。
   2. 浏览器检查证书是否被吊销，可能通过访问 CA 的证书撤销列表（CRL）或使用在线证书状态协议（OCSP）。
   3. 浏览器确认证书对应的域名与正在访问的网站域名匹配，并检查证书是否在有效期内。
3. 浏览器生成预主密钥并加密
   1. 浏览器生成一个预主密钥（pre-master secret）。
   2. 使用服务器的公钥加密预主密钥，并将其发送给服务器。
4. 服务器解密并建立会话密钥
   1. 服务器使用其私钥解密收到的预主密钥。
   2. 服务器和浏览器使用预主密钥和之前交换的随机数生成会话密钥。
   3. 使用这个会话密钥，服务器和浏览器可以安全地加密通信数据。
5. 完成握手
   1. 客户端和服务器交换加密的“Finished”消息，确认握手过程的完成，开始加密通信。

## 3 购买一个证书

购买证书前需要在服务端使用 openssl 生成公钥和私钥，填写域名和组织信息并用私钥签名后发送到 CA 机构进行认证。认证完成后 CA 机构会返回一个证书，证书安装到服务端后即可开通 SSL 服务。具体步骤如下：

1. 在生成密钥对

   ```
   openssl genrsa -out private.key 2048
   ```

2. 创建证书签名请求

   使用私钥创建一个 CSR。CSR 包含您的域名和组织信息，并且需要您的私钥来签名。输入下面的命令，并且按照提示填写组织信息。

   ```\
   openssl req -new -key private.key -out certificate.csr
   ```

3. 选择并提交到证书颁发机构(CA)

   打开 CA 机构的证书购买页面，上传 `certificate.csr` ，CA 机构会审核文件中的域名、组织等信息。

4. 域名验证

   等 CA 机构审核通过之后，还需要验证域名的真实性。这个时候 CA 机构会要求你按照提示解析你的域名，当你设置好解析之后，CA 机构会自动进行验证。验证通过之后就会证书了。

5. 安装证书

   从 CA 机构下载证书后在服务端安装证书。

## 4 Nginx 介绍

Nginx是一款高性能的Web服务器和反向代理服务器，它以其高效、稳定、轻量级和易于配置而闻名。

### 4.1 无域名的监听

Nginx 可以监听服务器端口的请求，并且根据请求做出回应。如:

```
server {
    listen 80; # 监听80端口

    # 网站的根目录
    root /var/www/mywebsite;
    # SSL证书路径
    ssl_certificate /path/to/your/fullchain.pem;
    ssl_certificate_key /path/to/your/privkey.key;

    # 默认页面
    index index.html index.htm;
}

```

上面的例子中，ngnix 会监听服务器的 80 端口, 并且将 80 端口的请求引导到本地的 /var/www/mywebsite/index.html 页面。

`/path/to/your/fullchain.pem` 和 `/path/to/your/privkey.key` SSL 证书的购买申请通过 CA 机构审核之后从购买网站下载得到的。

### 4.2 有域名的监听

如果你有两个域名，这两个域名对应两个不同的站点，但是这两个站点又在同一个服务器上，这时候就需要用到 ngnix 的多站点映射功能了：

```
server {
    listen 80;
    server_name example1.com www.example1.com;

    # example1.com的配置
    root /var/www/example1;
    index index.html;
    # ...其他设置...
}

server {
    listen 80;
    server_name example2.com www.example2.com;

    # example2.com的配置
    root /var/www/example2;
    index index.html;
    # ...其他设置...
}
```

### 4.3 反向代理

如果你在服务器A 上运行了一个 web 服务，对应的 ipA。为了服务器的安全，你想隐藏ipA，你可以在IP地址为ipB的服务器B上进行反向代理。此时你访问ipB时，服务器 B 会将请求转送给服务器 A，并在接收到服务器A 的请求后代替服务器 A 向客服端返回请求。如：

 ```
server {
    listen 80;
    # server_name是可选项
    # server_name example.com;

    location / {
        proxy_pass ipA:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
 ```



