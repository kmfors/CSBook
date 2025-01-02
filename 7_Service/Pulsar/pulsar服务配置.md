# pulsar服务配置

## 鉴权认证

参考文献：

https://blog.csdn.net/bouncing_/article/details/125494614

https://pulsar.apache.org/docs/4.0.x/security-token-admin/

https://blog.csdn.net/m0_59685732/article/details/139633539

### JWT 认证

Pulsar 基于 [JSON Web Tokens](https://jwt.io/introduction/) ([RFC-7519](https://tools.ietf.org/html/rfc7519)) 提供了一个标准的 JWT 客户端，可以生成密钥、制作 token 和检测密钥有效性等功能。

一个JWT的token字符串包含三个部分，彼此之间通过点来分割：

- Header 签名算法
- Payload 用户名与过期时间
- Signature 以特定算法生成的加密值

`bin/pulsar tokens show`: 查看 token 的 header 和 payload：

```shell
bin/pulsar tokens show -i "3NzYwOTh9.awbp6DreQwUyV8UCkYyOGXCFbfo4ZoV-dofXYTnFXO8"

{"alg":"HS256"}
---
{"sub":"test-user","exp":1656776098}
```

token 参数的配置既可使用字符串形式，也可使用文件形式，可择优选择：

```shell
# 字符串形式
brokerClientAuthenticationParameters={"token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0LXVzZXI"}

# 从文件读取
brokerClientAuthenticationParameters={"file":"///path/to/proxy-token.txt"}
```

​	

### 使用对称密钥

#### 生成 secret key

对称密钥，保存到文件 `my-secret.key` 中，后续将使用这个密钥文件来创建 token。

```shell
# 二进制编码，不可见
bin/pulsar tokens create-secret-key --output my-secret.key

# base64编码，明文可见
bin/pulsar tokens create-secret-key --output my-secret.key --base64
```

#### 生成用于超级管理员的 token

我们将超级管理员命名为 admin（对应 pulsar 认证概念里对 role）。这里不指定超时时间（ `--expiry-time`），则默认将不过期。

```shell
bin/pulsar tokens create --secret-key my-secret.key --subject admin


eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.Ra9pwWHTWjB67v5GkVuuDMqXWwfeTJuwflyvmhxYk_c
```

#### 生成给测试用户的 token

```shell
bin/pulsar tokens create --secret-key my-secret.key --subject test-user --expiry-time 7d

# 生成结果：
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0LXVzZXIiLCJleHAiOjE2NTY3NzYwOTh9.awbp6DreQwUyV8UCkYyOGXCFbfo4ZoV-dofXYTnFXO8
```

#### 配置 broker

```shell
# 开启认证
authenticationEnabled=true

# 认证提供者
authenticationProviders=org.apache.pulsar.broker.authentication.AuthenticationProviderToken

# 开启授权
authorizationEnabled=true

# 超级管理员
superUserRoles=admin

# broker Client 使用等认证插件
brokerClientAuthenticationPlugin=org.apache.pulsar.client.impl.auth.AuthenticationToken

# broker Client 通讯使用的 token（需要 admin role）
brokerClientAuthenticationParameters={"token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.Ra9pwWHTWjB67v5GkVuuDMqXWwfeTJuwflyvmhxYk_c"}

# 使用 secretKey 的密钥文件位置（file://开头）
tokenSecretKey=file:///Users/futeng/workspace/github/futeng/pulsar-pseudo-cluster/pulsar-1/my-secret.key
```

#### 重启 broker

```shell
bin/pulsar-daemon stop broker
bin/pulsar-daemon start broker
```

#### 测试

**验证 broker token**，注意 broker 的 token 需要使用超级管理员。

```shell
bin/pulsar tokens validate -sk  y-secret.key -i "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.Ra9pwWHTWjB67v5GkVuuDMqXWwfeTJuwflyvmhxYk_c"

# 打印：{sub=admin}
```

**测试超级管理员用户访问**

```shell
# produce as admin role
bin/pulsar-client \
--url "pulsar://127.0.0.1:6650" \
--auth-plugin "org.apache.pulsar.client.impl.auth.AuthenticationToken" \
--auth-params {"token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.Ra9pwWHTWjB67v5GkVuuDMqXWwfeTJuwflyvmhxYk_c"} \
produce public/default/test -m "hello pulsar" -n 10
```

**测试普通用户访问**

```shell
# produce as test-user role
bin/pulsar-client \
--url "pulsar://127.0.0.1:6650" \
--auth-plugin "org.apache.pulsar.client.impl.auth.AuthenticationToken" \
--auth-params {"token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0LXVzZXIiLCJleHAiOjE2NTY3NzYwOTh9.awbp6DreQwUyV8UCkYyOGXCFbfo4ZoV-dofXYTnFXO8"} \
produce public/default/test -m "hello pulsar" -n 10
```

> [!NOTE]
>
> 尚未给普通用户赋权，因此命令执行需要报缺少权限的错误。

**测试给普通用户赋权**

```shell
bin/pulsar-admin \
--admin-url "http://127.0.0.1:8080/" \
--auth-params {"token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.Ra9pwWHTWjB67v5GkVuuDMqXWwfeTJuwflyvmhxYk_c"} \
--auth-plugin "org.apache.pulsar.client.impl.auth.AuthenticationToken" \
namespaces grant-permission public/default --role test-user --actions produce,consume
```

- 赋权后，可再次测试普通用户访问，需要可以正常发送数据。
- 注意 `bin/pulsar-admin` 命令默认使用的是管理流 8080 端口。
- 注意 `auth-params` 需要使用超级管理员 token 来执行赋权。

**测试给普通用户回收权限**

```shell
bin/pulsar-admin \
--admin-url "http://127.0.0.1:8080/" \
--auth-params {"token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.Ra9pwWHTWjB67v5GkVuuDMqXWwfeTJuwflyvmhxYk_c"} \
--auth-plugin "org.apache.pulsar.client.impl.auth.AuthenticationToken" \ 
namespaces revoke-permission public/default --role test-user
```



### 使用非对称密钥

#### 生成秘钥对

```shell
# 二进制编码，不可见
bin/pulsar tokens create-key-pair --output-private-key my-private.key --output-public-key my-public.key
```

#### 生成用于超级管理员的 token

我们将超级管理员命名为 admin（对应 pulsar 认证概念里对 role）。这里不指定超时时间（ `--expiry-time`），则默认将不过期。

```shell
bin/pulsar tokens create --private-key my-private.key --subject admin

eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.ijp-Qw4JDn1aOQbYy4g4YGBbXYIgLA9lCVrnP-heEtPCdDq11_c-9pQdQwc6RdphvlSfoj50qwL5OtmFPysDuF2caSYzSV1kWRWN-tFzrt-04_LRN-vlgb6D06aWubVFJQBC4DyS-INrYqbXETuxpO4PI9lB6lLXo6px-SD5YJzQmcYwi2hmQedEWszlGPDYi_hDG9SeDYmnMpXTtPU3BcjaDcg9fO6PlHdbnLwq2MfByeIj-VS6EVhKUdaG4kU2EJf5uq2591JJAL5HHiuTZRSFD6YbRXuYqQriw4RtnYWSvSeVMMbcL-JzcSJblNbMmIOdiez43MPYFPTB7TMr8g
```

#### 生成给测试用户的 token

```shell
# d是天，y是月
bin/pulsar tokens create --private-key my-private.key --subject test-user --expiry-time 7d

eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ0ZXN0LXVzZXIiLCJleHAiOjE2NTY4MDMzODh9.0dAXdyl1dVsLZbhnvJDKPXFmyNlqwDYMMwzOoJ1L2Rl9gfcgVB4DzEfBFesU1F07P5oiM_X5hmxdI5YDSDxU4VGb_Sy3MakOAlROq3a4qzT45eY15-N3IxyfaI66BellDsZWyXVwsWnPYmwMBOlqZXgZAEhPL8HqC3c1IMBeMo78lDNobP7k0SVWsy9jhhmVOcas2ZQ4B-vOC8f0pHAWD29Rf_AV34A5w6Wu5XbQoHpMp5n0KRv2K_oFed_Zmg79uvtLv3Ujd8DaXN9a2vjXRatFYY2iZN8OhB1SV4WjpXB5hyG5Sv9uAHC559W39g8-AznG8NA5J79d-tIftIr8Dg
```

#### 配置 broker

```shell
# 开启认证
authenticationEnabled=true

# 认证提供者
authenticationProviders=org.apache.pulsar.broker.authentication.AuthenticationProviderToken

# 开启授权
authorizationEnabled=true

# 超级管理员
superUserRoles=admin

# broker Client 使用等认证插件
brokerClientAuthenticationPlugin=org.apache.pulsar.client.impl.auth.AuthenticationToken

# broker Client 通讯使用的 token（需要 admin role）
brokerClientAuthenticationParameters={"token":"eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.ijp-Qw4JDn1aOQbYy4g4YGBbXYIgLA9lCVrnP-heEtPCdDq11_c-9pQdQwc6RdphvlSfoj50qwL5OtmFPysDuF2caSYzSV1kWRWN-tFzrt-04_LRN-vlgb6D06aWubVFJQBC4DyS-INrYqbXETuxpO4PI9lB6lLXo6px-SD5YJzQmcYwi2hmQedEWszlGPDYi_hDG9SeDYmnMpXTtPU3BcjaDcg9fO6PlHdbnLwq2MfByeIj-VS6EVhKUdaG4kU2EJf5uq2591JJAL5HHiuTZRSFD6YbRXuYqQriw4RtnYWSvSeVMMbcL-JzcSJblNbMmIOdiez43MPYFPTB7TMr8g"}

# 使用 tokenPublicKey 的公钥文件位置（file://开头）
tokenPublicKey=file:///Users/futeng/workspace/github/futeng/pulsar-pseudo-cluster/pulsar-1/my-public.key
```

#### 重启 broker

```shell
bin/pulsar-daemon stop broker
bin/pulsar-daemon start broker
```

#### 测试

**验证 broker token**

```shell
bin/pulsar tokens validate -pk  /Users/futeng/workspace/github/futeng/pulsar-pseudo-cluster/pulsar-1/my-public.key -i "eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.ijp-Qw4JDn1aOQbYy4g4YGBbXYIgLA9lCVrnP-heEtPCdDq11_c-9pQdQwc6RdphvlSfoj50qwL5OtmFPysDuF2caSYzSV1kWRWN-tFzrt-04_LRN-vlgb6D06aWubVFJQBC4DyS-INrYqbXETuxpO4PI9lB6lLXo6px-SD5YJzQmcYwi2hmQedEWszlGPDYi_hDG9SeDYmnMpXTtPU3BcjaDcg9fO6PlHdbnLwq2MfByeIj-VS6EVhKUdaG4kU2EJf5uq2591JJAL5HHiuTZRSFD6YbRXuYqQriw4RtnYWSvSeVMMbcL-JzcSJblNbMmIOdiez43MPYFPTB7TMr8g"

# 打印：{sub=admin}
```

> [!NOTE]
>
> - 注意 broker 的 token 需要使用超级管理员。
> - 注意是使用公钥来验证 token（public.key）。

**测试超级管理员用户访问**

```shell
# produce as admin role
bin/pulsar-client \
--url "pulsar://127.0.0.1:6650" \
--auth-plugin "org.apache.pulsar.client.impl.auth.AuthenticationToken" \
--auth-params {"token":"eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.ijp-Qw4JDn1aOQbYy4g4YGBbXYIgLA9lCVrnP-heEtPCdDq11_c-9pQdQwc6RdphvlSfoj50qwL5OtmFPysDuF2caSYzSV1kWRWN-tFzrt-04_LRN-vlgb6D06aWubVFJQBC4DyS-INrYqbXETuxpO4PI9lB6lLXo6px-SD5YJzQmcYwi2hmQedEWszlGPDYi_hDG9SeDYmnMpXTtPU3BcjaDcg9fO6PlHdbnLwq2MfByeIj-VS6EVhKUdaG4kU2EJf5uq2591JJAL5HHiuTZRSFD6YbRXuYqQriw4RtnYWSvSeVMMbcL-JzcSJblNbMmIOdiez43MPYFPTB7TMr8g"} \
produce public/default/test -m "hello pulsar" -n 10
```

**测试普通用户访问**

```shell
# produce as test-user role
bin/pulsar-client \
--url "pulsar://127.0.0.1:6650" \
--auth-plugin "org.apache.pulsar.client.impl.auth.AuthenticationToken" \
--auth-params {"token":"eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ0ZXN0LXVzZXIiLCJleHAiOjE2NTY4MDMzODh9.0dAXdyl1dVsLZbhnvJDKPXFmyNlqwDYMMwzOoJ1L2Rl9gfcgVB4DzEfBFesU1F07P5oiM_X5hmxdI5YDSDxU4VGb_Sy3MakOAlROq3a4qzT45eY15-N3IxyfaI66BellDsZWyXVwsWnPYmwMBOlqZXgZAEhPL8HqC3c1IMBeMo78lDNobP7k0SVWsy9jhhmVOcas2ZQ4B-vOC8f0pHAWD29Rf_AV34A5w6Wu5XbQoHpMp5n0KRv2K_oFed_Zmg79uvtLv3Ujd8DaXN9a2vjXRatFYY2iZN8OhB1SV4WjpXB5hyG5Sv9uAHC559W39g8-AznG8NA5J79d-tIftIr8Dg"} \
produce public/default/test -m "hello pulsar" -n 10
```

> [!NOTE]
>
> 尚未给普通用户赋权，因此命令执行需要报缺少权限的错误。

**测试给普通用户赋权**

```shell
bin/pulsar-admin \
--admin-url "http://127.0.0.1:8080/" \
--auth-params {"token":"eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.ijp-Qw4JDn1aOQbYy4g4YGBbXYIgLA9lCVrnP-heEtPCdDq11_c-9pQdQwc6RdphvlSfoj50qwL5OtmFPysDuF2caSYzSV1kWRWN-tFzrt-04_LRN-vlgb6D06aWubVFJQBC4DyS-INrYqbXETuxpO4PI9lB6lLXo6px-SD5YJzQmcYwi2hmQedEWszlGPDYi_hDG9SeDYmnMpXTtPU3BcjaDcg9fO6PlHdbnLwq2MfByeIj-VS6EVhKUdaG4kU2EJf5uq2591JJAL5HHiuTZRSFD6YbRXuYqQriw4RtnYWSvSeVMMbcL-JzcSJblNbMmIOdiez43MPYFPTB7TMr8g"} \
--auth-plugin "org.apache.pulsar.client.impl.auth.AuthenticationToken" \
namespaces grant-permission public/default --role test-user --actions produce,consume
```

- 赋权后，可再次测试普通用户访问，需要可以正常发送数据。
- 注意 `bin/pulsar-admin` 命令默认使用的是管理流 8080 端口。
- 注意 `auth-params` 需要使用超级管理员 token 来执行赋权。

**测试给普通用户回收权限**

```shell
bin/pulsar-admin \
--admin-url "http://127.0.0.1:8080/" \
--auth-params {"token":"eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9.ijp-Qw4JDn1aOQbYy4g4YGBbXYIgLA9lCVrnP-heEtPCdDq11_c-9pQdQwc6RdphvlSfoj50qwL5OtmFPysDuF2caSYzSV1kWRWN-tFzrt-04_LRN-vlgb6D06aWubVFJQBC4DyS-INrYqbXETuxpO4PI9lB6lLXo6px-SD5YJzQmcYwi2hmQedEWszlGPDYi_hDG9SeDYmnMpXTtPU3BcjaDcg9fO6PlHdbnLwq2MfByeIj-VS6EVhKUdaG4kU2EJf5uq2591JJAL5HHiuTZRSFD6YbRXuYqQriw4RtnYWSvSeVMMbcL-JzcSJblNbMmIOdiez43MPYFPTB7TMr8g"} \
--auth-plugin "org.apache.pulsar.client.impl.auth.AuthenticationToken" \
namespaces revoke-permission public/default --role test-user 
```

### 配合使用 TLS 传输加密

可参考：https://pulsar.apache.org/docs/security-tls-transport