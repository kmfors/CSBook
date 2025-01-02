# Pulsar 组件

pulsar服务：apache-pulsar-3.2.1-bin.tar.gz

pulsar可视化：apache-pulsar-manager-0.4.0-bin.tar.gz（所需jdk8，node）

pulsar客户端：apache-pulsar-client-cpp-3.4.2.tar.gz

[pulsar组件下载](https://pulsar.apache.org/download/)

https://blog.csdn.net/CallousZD/article/details/128818250

```shell
#前端单机启动 
bin/pulsar standalone 
# 后台单机启动 
bin/pulsar-daemon start standalone
```


```shell
cp -r ../dist/ ui/
# 前端启动pulsar-manager
bin/pulsar-manager

# 后台启动pulsar-manager
nohup bin/pulsar-manager > pulsar-manager.log 2>&1 &

# 增加管理员账号
CSRF_TOKEN=$(curl http://192.168.20.10:7750/pulsar-manager/csrf-token)
curl \
   -H 'X-XSRF-TOKEN: $CSRF_TOKEN' \
   -H 'Cookie: XSRF-TOKEN=$CSRF_TOKEN;' \
   -H "Content-Type: application/json" \
   -X PUT http://192.168.20.10:7750/pulsar-manager/users/superuser \
   -d '{"name": "pulsar", "password": "pulsar", "description": "dev", "email": "maskic@foxmail.com"}'

CSRF_TOKEN=$(curl http://106.54.5.6:7750/pulsar-manager/csrf-token)
curl \
   -H 'X-XSRF-TOKEN: $CSRF_TOKEN' \
   -H 'Cookie: XSRF-TOKEN=$CSRF_TOKEN;' \
   -H "Content-Type: application/json" \
   -X PUT http://106.54.5.6:7750/pulsar-manager/users/superuser \
   -d '{"name": "pulsar", "password": "pulsar", "description": "dev", "email": "maskic@foxmail.com"}'

```

将dist包拷贝到pulsar-manager目录下并更名为ui  

启动pulsar-manager，**注意：启动命令一定要在pulsar-manager的安装目录下执行，否则访问时，会发生ui目录找不到的问题**

http://192.168.203.101:8080


Environment Name:	pulsar-cluster
Service URL:	http://192.168.203.101:8080
Bookie URL:		http://192.168.203.101:2181,192.168.203.102:2181,192.168.203.103:2181

# pulsar客户端

1、vcpkg的安装（cmake -B build -DINTEGRATE_VCPKG=ON ,可取消）

2、openssl的安装

3、pkg-config 安装：手动安装 **sudo apt install pkg-config**

4、boost库安装