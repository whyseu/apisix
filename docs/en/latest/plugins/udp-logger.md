---
title: udp-logger
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

## Name

`udp-logger` is a plugin which push Log data requests to UDP servers.

This will provide the ability to send Log data requests as JSON objects to Monitoring tools and other UDP servers.

This plugin provides the ability to push Log data as a batch to you're external UDP servers. In case if you did not receive the log data don't worry give it some time it will automatically send the logs after the timer function expires in our Batch Processor.

For more info on Batch-Processor in Apache APISIX please refer.
[Batch-Processor](../batch-processor.md)

## Attributes

| Name             | Type    | Requirement | Default      | Valid   | Description                                                                              |
| ---------------- | ------- | ----------- | ------------ | ------- | ---------------------------------------------------------------------------------------- |
| host             | string  | required    |              |         | IP address or the Hostname of the UDP server.                                            |
| port             | integer | required    |              | [0,...] | Target upstream port.                                                                    |
| timeout          | integer | optional    | 3            | [1,...] | Timeout for the upstream to send data.                                                   |
| name             | string  | optional    | "udp logger" |         | A unique identifier to identity the batch processor                                      |
| include_req_body | boolean | optional    | false        |         | Whether to include the request body                                                      |

The plugin supports the use of batch processors to aggregate and process entries(logs/data) in a batch. This avoids frequent data submissions by the plugin, which by default the batch processor submits data every `5` seconds or when the data in the queue reaches `1000`. For information or custom batch processor parameter settings, see [Batch-Processor](../batch-processor.md#configuration) configuration section.

## How To Enable

The following is an example on how to enable the udp-logger for a specific route.

```shell
curl http://127.0.0.1:9080/apisix/admin/routes/5 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
      "plugins": {
            "udp-logger": {
                 "host": "127.0.0.1",
                 "port": 3000,
                 "batch_max_size": 1,
                 "name": "udp logger"
            }
       },
      "upstream": {
           "type": "roundrobin",
           "nodes": {
               "127.0.0.1:1980": 1
           }
      },
      "uri": "/hello"
}'
```

## Test Plugin

* success:

```shell
$ curl -i http://127.0.0.1:9080/hello
HTTP/1.1 200 OK
...
hello, world
```

## Disable Plugin

Remove the corresponding json configuration in the plugin configuration to disable the `udp-logger`.
APISIX plugins are hot-reloaded, therefore no need to restart APISIX.

```shell
$ curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "methods": ["GET"],
    "uri": "/hello",
    "plugins": {},
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```
