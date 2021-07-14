---
title: 证书
---

<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

`APISIX` 支持通过 TLS 扩展 SNI 实现加载特定的 SSL 证书以实现对 https 的支持。

SNI(Server Name Indication)是用来改善 SSL 和 TLS 的一项特性，它允许客户端在服务器端向其发送证书之前向服务器端发送请求的域名，服务器端根据客户端请求的域名选择合适的SSL证书发送给客户端。

## 一：配置

执行脚本如下 ->

```shell
curl --location --request PUT 'http://127.0.0.1:9080/apisix/admin/ssl/1' \
--header 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \
--header 'Content-Type: application/json' \
--data '{
    "key":"xxx.key 文件内容",
    "cert":"xxx.pem 文件内容",
    "snis":["你希望配置的可访问域名"]
  }'
```


```html
参数说明：
X-API-KEY: config-default.yml 里边的admin_key.name.key值，注意要取admin的，egg:
   admin_key:
      name: "admin"
      key: edd1c9f034335f136f87ad84b625c8f1

key ： 证书 xxx.key 文件内容
cert:  证书 xxx.pem 文件内容
snis： 可访问的指定域名，注意需要跟证书保持一致，比如我们的证书支持 *.winnermedical.com,不符合这个条件的域名证书不会去支持
```


证书参考：
xxx__winnermedical.com.key
xxx__winnermedical.com.pem


由于直接把证书内容复制进去，curl无法正常访问，存在转义字符问题，这里附上对应的转义脚本
```java
package com.purcotton.openapi.test.apisxi.ssl;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

/**
 * @author wangkai
 * @date 2021/7/13
 * @since 1.0.0
 **/
public class Test {
    public static void main(String[] args) throws JsonProcessingException {
        String key = "粘贴上完整的xxx.key文件内容即可";

        String cert = "粘贴上完整的xxx.pem文件内容即可";

        String[] snis = {"xxx-test.winnermedical.com","xxx2-test.winnermedical.com"};
        TestRequest request = new TestRequest();
        request.setCert(cert);
        request.setKey(key);
        request.setSnis(snis);
        ObjectMapper mapper = new ObjectMapper();
        String s = mapper.writeValueAsString(request);
        System.out.println(s);
    }

    private static class TestRequest{
        private String key;
        private String cert;
        private String[] snis;

        public String getKey() {
            return key;
        }

        public void setKey(String key) {
            this.key = key;
        }

        public String getCert() {
            return cert;
        }

        public void setCert(String cert) {
            this.cert = cert;
        }

        public String[] getSnis() {
            return snis;
        }

        public void setSnis(String[] snis) {
            this.snis = snis;
        }
    }
}

```

把执行的结果，替换下面脚本的 "请把执行结果直接粘贴到这里"

```shell
curl --location --request PUT 'http://127.0.0.1:9080/apisix/admin/ssl/1' \
--header 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \
--header 'Content-Type: application/json' \
--data '请把执行结果直接粘贴到这里'
```

然后把上面的脚本，粘贴到网关的机子上运行，注意只能是127.0.0.1环境


执行完毕，在etcd的/apisix/ssl/1就能找到对应的记录，直接成功之后默认整个网关都具备https协议的能力了，注意只能是在9443这个端口（可根据需要修改这个端口,在config-default.yml文件），egg:

```java
 ssl:
    enable: true
    enable_http2: true
    listen_port: 9443
```

另外，需要特别注意的是，不是说任何情况都能使用9443这个端口，请求的客户端原始地址必须是 上面 snis 参数指定的域名才可以，比如，以下请求：

```java
curl --location --request GET 'https://xxx-test.winnermedical.com/get' \
--header 'x-winner-app-id: winner.demo' \
--header 'x-winner-env: test' \
--header 'Cookie: BD_HOME=1; BDSVRTM=14'
```

因为客户端请求的域名 xxx-test.winnermedical.com 在 snis 里边，所以能正常使用https功能
