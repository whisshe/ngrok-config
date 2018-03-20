# 2018-03-19
## ngrok简介
  ngrok是一个开源的内网穿透服务(1.7之前的版本)，通过反向代理实现端口间的映射，使得内网服务(内网中所有的机器)能够通过外网IP/域名进行访问(将请求转发至指定机器，内网中安装一个客户端即可)。
## 必备条件
1. 一台有外网IP的可提供服务的服务器(用来运行ngrok服务端)
2. GO语言环境
3. 开放防火墙端口
## ngrok服务端
[ngrok github地址](https://github.com/inconshreveable/ngrok.git)
GitHub上面提供的是服务端的源码，客户端是在指定配置之后编译生成的。
1. 安装GO语言环境
2. 安装ngrok，编译生成客户端
   ```
      git clone  https://github.com/inconshreveable/ngrok.git
      cd ngrok
   ```
   #### 配置证书信息，以生成专属的客户端(可写入shell脚本执行)
   ```   
     NGROK_DOMAIN="abc.com" #换成你的域名(绑定外网IP的域名)

     openssl genrsa -out base.key 2048
     openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out base.pem
     openssl genrsa -out server.key 2048
     openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
     openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt

     cp base.pem assets/client/tls/ngrokroot.crt
   ```
   #### 编译生成客户端和服务端(有针对不同系统生成的客户端，自行搜索)
   ```
     make release-server release-client
   ```
   #### 编译生成的执行文件放在ngrok/bin下，ngrok为客户端，ngrokd为服务端。
3. ngrok-server参数说明
   ```
    -httpAddr  # http端口(通过外网IP或域名加上这个端口进行http访问内网)
    -httpsAddr #同http，只是这个是https
    -tunnelAddr #隧道端口，内网和外网建立的隧道端口，默认为4443(客户端配置连接的端口)
   ```
   #### ngrokd启动命令
   ```
   nohup ./bin/ngrokd -tlsKey=server.key -tlsCrt=server.crt -domain="your domain" -httpAddr=":8081"  &
   #nohup可将进程置入后台，且ssh连接断开不受影响
   netstat -tpln|grep ngrokd  #查看进程端口状态
   ```
## ngrok客户端
1. 将客户端从外网服务器上面下载至内网机器
2. 在内网机器上写一个配置文件ngrok.cfg
   #### 该配置文件可一次性转发多个端口，建议按这种格式，配置文件为yaml语法，所有缩进需要使用空格
   ```
   server_addr: ngrok.abc.com:4443
   trust_host_root_certs: false
   tunnels:
    ssh:
     remote_port: 1122
     proto:
      tcp: 22
    shadowsock:
     emote_port: 1088
     proto:
      tcp: 1188
    ftp:
     remote_port: 24
     proto:
      tcp: 24
    http:
     subdomain: gitlab
     proto:
      http: 80
      https: 172.3.2.1:443
    ```
    #### 启动特定的隧道
    ```
    ./ngrok -config ngrok.cfg start ssh ftp
    ```
    #### 启用配置文件中所有的隧道
    ```
    ./ngrok -config ngrok.cfg start-all
    ```
