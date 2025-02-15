---
title: referer-restriction
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

## 名字

`referer-restriction` 插件可以根据 Referer 请求头限制访问。

## 属性

| 参数名    | 类型          | 可选项 | 默认值 | 有效值 | 描述                             |
| --------- | ------------- | ------ | ------ | ------ | -------------------------------- |
| whitelist | array[string] | 可选    |         |       | 白名单域名列表。域名开头可以用'*'作为通配符。 |
| blacklist | array[string] | 可选    |         |       | 黑名单域名列表。域名开头可以用'*'作为通配符。 |
| message | string | 可选    | Your referer host is not allowed | [1, 1024] | 在未允许访问的情况下返回的信息。 |
| bypass_missing  | boolean       | 可选    | false   |       | 当 Referer 不存在或格式有误时，是否绕过检查。 |

只能单独启用白名单或黑名单，两个不能一起使用。
`message`可以由用户自定义。

## 如何启用

下面是一个示例，在指定的 route 上开启了 `referer-restriction` 插件:

```shell
curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index.html",
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    },
    "plugins": {
        "referer-restriction": {
            "bypass_missing": true,
            "whitelist": [
                "xx.com",
                "*.xx.com"
            ]
        }
    }
}'
```

## 测试插件

带 `Referer: http://xx.com/x` 请求:

```shell
$ curl http://127.0.0.1:9080/index.html -H 'Referer: http://xx.com/x'
HTTP/1.1 200 OK
...
```

带 `Referer: http://yy.com/x` 请求:

```shell
$ curl http://127.0.0.1:9080/index.html -H 'Referer: http://yy.com/x'
HTTP/1.1 403 Forbidden
...
{"message":"Your referer host is not allowed"}
```

不带 `Referer` 请求:

```shell
$ curl http://127.0.0.1:9080/index.html
HTTP/1.1 200 OK
...
```

## 禁用插件

当你想去掉 `referer-restriction` 插件的时候，很简单，在插件的配置中把对应的 json 配置删除即可，无须重启服务，即刻生效：

```shell
$ curl http://127.0.0.1:9080/apisix/admin/routes/1  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index.html",
    "plugins": {},
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```

现在就已移除 `referer-restriction` 插件，其它插件的开启和移除也类似。
