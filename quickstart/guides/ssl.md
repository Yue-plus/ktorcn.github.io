---
title: SSL
caption: 如何获取免费证书以及在 Ktor 中使用 SSL
category: quickstart
keywords: tls ssl https let's encrypt letsencrypt
---

{::options toc_levels="1..2" /}
{% include nomnoml-support.html %}

![](/quickstart/guides/ssl/lets-encrypt.svg)

**目录：**

* TOC
{:toc}

可以购买证书并配置 Ktor 使用，
**也可以**使用 Let's Encrypt 自动获取 **免费证书** 来以 Ktor 提供支持 `https://` 与 `wss://` 请求<!--
-->的服务。
本页会介绍如何实现，既有通过直接配置 Ktor 以提供单个域名的 SSL 证书的服务，
也有使用 Docker 与 nginx 轻松在一台计算机上为位于不同计算机的不同应用提供服务<!--
-->。

## 选项一：直接以 Ktor 提供 SSL 服务
{: #ktor}

### 配置 `一个`指向机器的注册点

首先，必须将域名或者子域名配置为指向即将用于证书的<!--
-->计算机的 IP。 必须在这里使用计算机的公网 IP。
如果该计算机位于路由器之后，那么需要配置路由器将该计算机与对应主机名做 DMZ，
或者至少将 80 端口（HTTP）重定向到该计算机，而之后可能也需要配置
443 端口（HTTPS）。

Let's Encrypt 总是访问你的公网 IP 的 80 端口，即便你配置 Ktor 绑定了其他端口，
你必须配置路由将 80 端口重定向到运行 ktor 的计算机的正确本机 IP 与端口<!--
-->。
{: .note }

### 生成证书

Ktor 服务器必须没有运行，然后必须运行以下命令
（需更改 `my.example.com`、 `root@example.com` 与 `8889`）

这个命令会在指定的端口启动一个 HTTP 服务器（该端口必须在公网暴露为 80
端口，也可以在路由器转发端口 80:8889，并且域名必须指向你的公网 IP），
它会请求一个 challenge、以正确的内容暴露为 `/.well-known/acme-challenge/file`，
生成一个域名私钥并接收证书链：

```
export DOMAIN=my.example.com
export EMAIL=root@example.com
export PORT=8889
export ALIAS=myalias
certbot certonly -n -d $DOMAIN --email "$EMAIL" --agree-tos --standalone --preferred-challenges http --http-01-port $PORT
```

<table>
<tr>
<td markdown="1" style="width:50%;">
❌ 错误输出样例：

```text
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for my.example.com
Waiting for verification...
Cleaning up challenges
Failed authorization procedure. my.example.com (http-01): urn:acme:error:connection :: The server could not connect to the client to verify the domain :: Fetching http://my.example.com/.well-known/acme-challenge/j-BJXA5ZGXdJuZhTByL4B95XBpiaGjZsm8JdCcA3Vr4: Timeout during connect (likely firewall problem)

IMPORTANT NOTES:
 - The following errors were reported by the server:

   Domain: my.example.com
   Type:   connection
   Detail: Fetching
   http://my.example.com/.well-known/acme-challenge/j-BJXA5ZGXdJuZhTByL4B9zXBp3aGjZsm8JdCcA3Vr4:
   Timeout during connect (likely firewall problem)

   To fix these errors, please make sure that your domain name was
   entered correctly and the DNS A/AAAA record(s) for that domain
   contain(s) the right IP address. Additionally, please check that
   your computer has a publicly routable IP address and that no
   firewalls are preventing the server from communicating with the
   client. If you're using the webroot plugin, you should also verify
   that you are serving files from the webroot path you provided.
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
```
</td>
<td markdown="1" style="width:50%;">
✅ 正常输出样例：

```text
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for my.example.com
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/my.example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/my.example.com/privkey.pem
   Your cert will expire on 2018-09-27. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
</td>
</tr>
</table>

### 为 Ktor 转换私钥与证书

现在必须将 `certbot` 写入的私钥与证书转换为 Ktor 理解的格式。

证书链与私钥都存储在 `/etc/letsencrypt/live/$DOMAIN` 中，分别为 `fullchain.pem` 与 `privkey.pem`。

```
openssl pkcs12 -export -out /etc/letsencrypt/live/$DOMAIN/keystore.p12 -inkey /etc/letsencrypt/live/$DOMAIN/privkey.pem -in /etc/letsencrypt/live/$DOMAIN/fullchain.pem -name $ALIAS
```

导出时会要求设置密码（需要提供一个用于下一步工作的密码）：

```
Enter Export Password: mypassword
Verifying - Enter Export Password: mypassword
```

有了 p12 文件，可以使用 `keytool` 命令行工具来生成 JKS 文件：

```
keytool -importkeystore -alias $ALIAS -destkeystore /etc/letsencrypt/live/$DOMAIN/keystore.jks -srcstoretype PKCS12 -srckeystore /etc/letsencrypt/live/$DOMAIN/keystore.p12
```

### 配置 Ktor 使用生成的 JKS

现在必须更新 HOCON 文件 `application.conf`，配置 SSL 端口、keyStore、别名以及密码。
必须为你指定的场景设置正确的值：

```groovy
ktor {
    deployment {
        port = 8889
        port = ${?PORT}
        sslPort = 8890
        sslPort = ${?PORT_SSL}
    }
    application {
        modules = [ com.example.ApplicationKt.module ]
    }
    security {
        ssl {
            keyStore = /etc/letsencrypt/live/mydomain.com/keystore.jks
            keyAlias = myalias
            keyStorePassword = mypassword
            privateKeyPassword = mypassword
        }
    }
}
```

如果一切顺利，Ktor 应该是 HTTP 监听在 8889 端口而 HTTPS 监听在 8890 端口。

## 选择二：使用 Docker 并以 Nginx 作为反向代理
{: #docker}

使用具有多个域名的 Docker 时，可能希望使用 [nginx-proxy] 镜像以及 [letsencrypt-nginx-proxy-companion]
镜像在单个计算机/ip 中为多个域名/子域名提供服务，并使用 let’s encrypt 自动提供 HTTPS。

在这个场景中，创建一个带 NGINX 的容器，可能会监听 80 与 443 端口，内部网络<!--
-->只能在容器间访问，因此 nginx 可以连接并反向代理你的网站（包括 websocket），
而 NGINX 伴侣通过自省已配置的 Docker 容器来处理域名证书。

[nginx-proxy]: https://github.com/jwilder/nginx-proxy
[letsencrypt-nginx-proxy-companion]: https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion

### 创建一个 docker 内网

第一步是创建一个将会使用的桥接网络，以便 nginx 可以连接到其他容器<!--
-->以反向代理用户的 HTTP、HTTPS、WS 与 WSS 请求：

```bash
docker network create --driver bridge reverse-proxy
```

### 创建一个 Nginx 容器

现在应该创建一个运行 NGINX  的容器来进行反向代理：

```bash
docker rm -f nginx
docker run -d -p 80:80 -p 443:443 \
	--name=nginx \
	--restart=always \
	--network=reverse-proxy \
	-v /home/virtual/nginx/certs:/etc/nginx/certs:ro \
	-v /home/virtual/nginx/conf.d:/etc/nginx/conf.d \
	-v /home/virtual/nginx/vhost.d:/etc/nginx/vhost.d \
	-v /home/virtual/nginx/html:/usr/share/nginx/html \
	-v /var/run/docker.sock:/tmp/docker.sock:ro \
	-e NGINX_PROXY_CONTAINER=nginx \
	--label com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=true \
	jwilder/nginx-proxy
```

* `--restart=always` 以便计算机重启时，docker 守护进程重新启动容器。
* `--network=reverse-proxy` 以便 NGINX 在该网络中可以连接到同一网络中的其他容器。
* `-v certs:ro` 这个卷会与 letsencrypt-companion 共享以访问每个域名的证书。
* `-v conf, vhost` 这个配置是持久化且外部可访问的，以便应对必须调整配置的情况。
* `-v /var/run/docker.sock` 这允许本镜像获取关于在守护进程中运行的新容器的通知/自省。
* `-e --label` 供其伴侣使用，将本镜像标识为 NGINX。

可以将 `/home/virtual/nginx*` 路径调整为你选用的路径。

### 创建一个 Nginx Let's Encrypt 伴侣容器

有了 nginx-proxy 容器，现在可以创建一个伴侣容器，
它会请求与续订证书：

```bash
docker rm -f nginx-letsencrypt
docker run -d \
    --name nginx-letsencrypt \
    --restart=always \
    --network=reverse-proxy \
    --volumes-from nginx \
    -v /home/virtual/nginx/certs:/etc/nginx/certs:rw \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    jrcs/letsencrypt-nginx-proxy-companion
```

* `--restart=always` 作为 NGINX 镜像，在启动时重启。
* `--network=reverse-proxy` 它需要与 NGINX 代理容器位于同一网络以与之通信。
* `--volumes-from nginx` 这样可访问与 NGINX 容器相同的卷，因此可以在 `/usr/share/nginx/html` 内写入 `.well-known` challenge。
* `-v certs:rw` 它需要写访问权以写入私钥与证书供 NGINX 使用。
* `-v /var/run/docker.sock` 需要访问 docker 事件与自省来确定要请求的证书。

### 创建一个服务

现在已经配置了 NGINX 与 Let's Encrypt 伴侣，这样他们就会自动反向代理你的网站，
并根据环境变量 `VIRTUAL_HOST`、 `VIRTUAL_PORT` 以及 `LETSENCRYPT_HOST`、 `LETSENCRYPT_EMAIL` 为网站请求并提供证书。

使用 docker-compose，可以创建一个 `docker-compose.yml` 文件（而无其他服务）如下所示：

#### `docker-compose.yml`

```yaml
version: '2'
services:
  web:
    build:
      context: ./
      dockerfile: Dockerfile
    expose:
      - 8080
    environment:
      - VIRTUAL_HOST=mydomain.com
      - VIRTUAL_PORT=8080
      - LETSENCRYPT_HOST=mydomain.com
      - LETSENCRYPT_EMAIL=myemail@mydomain.com
    networks:
      - reverse-proxy
    restart: always

networks:
  backend:
  reverse-proxy:
    external:
      name: reverse-proxyv
```

#### `Dockerfile`

{% capture my_include %}{% include docker-sample.md %}{% endcapture %}
{{ my_include | markdownify }}

可以在部署一节找到关于 [how to deploy a docker and the Dockerfile](/servers/deploy.html#docker) 的更多信息。

### 简要概述

```nomnoml
#direction: right
#.internet: fill=#fee
#.network: fill=#efe
#.http: fill=#6f6
#.ssl: fill=#6af

[<internet>Internet]

[<http>Nginx
|port=80 (HTTP, WS)
|port=443 (HTTPS and WSS)
|TLS certs per domain
|VIRTUAL_HOST
|VIRTUAL_PORT
]

[App
|[port=8080 HTTP & WS]
|[<http>VIRTUAL_HOST=myhost.com]
|[<http>VIRTUAL_PORT=8080]
|[<ssl>LETSENCRYPT_HOST=myhost.com]
|[<ssl>LETSENCRYPT_EMAIL=email@myhost.com]
]

[<ssl>Let's Encrypt companion
|LETSENCRYPT_HOST
|LETSENCRYPT_EMAIL]

[Docker
|port=80,443
]

[Let's Encrypt] <- cert request [Let's Encrypt companion]

[App] -:> [reverse-proxy]

[<network>reverse-proxy|network]
[Nginx] <- [reverse-proxy]

[Internet] <- port 80, 443[Docker]
[Docker] <- [Nginx]

[Let's Encrypt companion] <-> [Nginx]
```
