---
title: ngrok
date: 2015-07-16 17:57:17
---

ngrok 是一款内网穿透神器，可以直接将内网的端口暴露到外网，实在是居家必备神器。

<!-- more -->

# 下载并安装 Go

```bash
cd ~/download
wget https://storage.googleapis.com/golang/go1.4.2.linux-amd64.tar.gz
tar -xvf  go1.4.2.linux-amd64.tar.gz
cp -R go /usr/local
cp /usr/local/go/bin/* /usr/bin
```

# 下载 ngrok

```bash
cd ~/github
git clone https://github.com/inconshreveable/ngrok.git
cd ngrok
```

# 开始编译 ngrok

- 首先为根域名生成证书

```bash
export NGROK_DOMAIN="yourdomain.com"
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=$NGROK_DOMAIN" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp device.crt assets/server/tls/snakeoil.crt
cp device.key assets/server/tls/snakeoil.key
```

- 编译服务器端

```
GOOS=linux GOARCH=amd64
#如果是32位系统，这里 GOARCH=386
```

然后 make

```
make release-server
```

- 编译 Windows 客户端

```
cd /usr/local/go/src
GOOS=windows GOARCH=amd64 ./make.bash
cd ~/github/ngrok
GOOS=windows GOARCH=amd64 make release-client
```

如果编译成功，应该在`bin/windows_amd64/`下产生一个`ngrok.exe`

# 运行

- 启动服务端

```
./ngrokd -domain="yourdomain.com" -httpAddr=":8080" -httpsAddr=":445"
```

- 启动客户端
  首先在 ngrok.exe 的同级目录新建一个配置文件 ngrok.conf

```
server_addr: "yourdomain.com:4443"
trust_host_root_certs: false
```

这里的 4443 是固定的，是 ngrok 在服务端监听的端口。

然后运行命令：

```
ngrok.exe -subdomain oms -config=ngrok.conf 8081
```
