# gonet

[![Build Status](https://travis-ci.org/bingoohuang/gonet.svg?branch=master)](https://travis-ci.org/bingoohuang/gonet)


net relative like port, http, rest.

1. FreePort 获得系统当前自由TCP端口（没有被占用）
1. Get/Post/Put/Patch/Delete HTTP 客户端调用
1. TLS relatives HTTPS证书
    
    * 根密钥/根证书生成
    * 服务端密钥/服务器证书生成
    * 客户端密钥/客户端证书生成
    * 服务端TLSConfig(https，客户端证书校验)
    * 客户端TLSConfig(服务器端证书校验，传递客户端证书)
    
1. ListLocalIfaceAddrs, ListLocalIps, ListLocalIPMap 列出本地IP及网卡名称
1. ReverseProxy 反向代理
1. IsLocalAddr 判断addr（ip，域名等）是否指向本机


## Make certs

参见[cert_test.sh](./cert_test.sh)

```bash
openssl genrsa -out root.key 2048
openssl req -new -nodes -x509 -days 3650 -key root.key -out root.pem -subj "/C=CN/ST=BEIJING/L=Earth/O=BJCA/OU=IT/CN=root"

openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr -subj "/C=CN/ST=BEIJING/L=Earth/O=BJCA/OU=IT/CN=server"
openssl x509 -req -in server.csr -CA root.pem -CAkey root.key -CAcreateserial -out server.pem -days 3650

openssl genrsa -out client.key 2048
openssl req -new -key client.key -subj "/C=CN/ST=BEIJING/L=Earth/O=BJCA/OU=IT/CN=client" -out client.csr
echo "[ssl_client]"> openssl.cnf; echo "extendedKeyUsage = clientAuth" >> openssl.cnf;
openssl x509 -req -in client.csr -CA root.pem -CAkey root.key -CAcreateserial -extfile ./openssl.cnf -out client.pem -days 3650
```

## go server usage demo

```go
import "github.com/bingoohuang/gonet"

// 传入服务端私钥文件，服务端证书文件，以及客户端根证书文件（可选 ，不传时不进行客户端证书校验）
tlsConfig := gonet.TLSConfigCreateServerMust(serverKeyFile, serverPemFile, clientRootPemFile)
addr := ":8080"
ln, err := tls.Listen("tcp", addr, tlsConfig)

route := gin.Default()
server := &http.Server{Addr: addr, Handler: route}
err := server.Serve(ln)
```

## go client usage demo

```go
import "github.com/bingoohuang/gonet"

// 传入客户端私钥文件，客户端证书文件，以及服务端根证书文件（可选 ，不传时不进行服务端证书校验）
tlsClientConfig := gonet.TLSConfigCreateClientMust(c.ClientKey, c.ClientPem, c.RootPem)
gonet.MustGet("https://httpbin.org/get").TLSClientConfig(tlsClientConfig).String()
```

具体客户端https证书使用案例，可以参见[typhon4g](https://github.com/bingoohuang/typhon4g)。


## Thanks

1. [urllib](https://github.com/GiterLab/urllib)
1. [go-resty](https://github.com/go-resty/resty/tree/v2)
1. [sling](https://github.com/dghubble/sling)
1. [This demonstrates how to make client side certificates with go](https://gist.github.com/ncw/9253562)
1. [带入gRPC：基于 CA 的 TLS 证书认证](https://studygolang.com/articles/15331)
1. [Better self-signed certificates](https://github.com/Shyp/generate-tls-cert)
1. [Create a PKI in GoLang](https://fale.io/blog/2017/06/05/create-a-pki-in-golang/)
1. [root CA and VerifyClientCert](https://play.golang.org/p/NyImQd5Xym)
1. [Golang的TLS证书](https://blog.csdn.net/fyxichen/article/details/51250620)
1. [密钥、证书生成和管理总结](https://www.cnblogs.com/pixy/p/4722381.html)
1. [TLS with Go](https://ericchiang.github.io/post/go-tls/)
1. [Golang - Go与HTTPS](http://www.golangtab.com/2018/02/05/Golang-Go与HTTPS/)
