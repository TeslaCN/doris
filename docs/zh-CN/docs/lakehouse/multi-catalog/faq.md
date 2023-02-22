---
{
    "title": "常见问题",
    "language": "zh-CN"
}
---

<!-- 
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->


# 常见问题

1. 通过 Hive Metastore 访问 Iceberg 表报错：`failed to get schema` 或 `Storage schema reading not supported`

	在 Hive 的 lib/ 目录放上 `iceberg` 运行时有关的 jar 包。

	在 `hive-site.xml` 配置：
	
	```
	metastore.storage.schema.reader.impl=org.apache.hadoop.hive.metastore.SerDeStorageSchemaReader
	```

	配置完成后需要重启Hive Metastore。

2. 连接 Kerberos 认证的 Hive Metastore 报错：`GSS initiate failed`

    通常是因为 Kerberos 认证信息填写不正确导致的，可以通过以下步骤排查：

    1. 1.2.1 之前的版本中，Doris 依赖的 libhdfs3 库没有开启 gsasl。请更新至 1.2.2 之后的版本。
    2. 确认对各个组件，设置了正确的 keytab 和 principal，并确认 keytab 文件存在于所有 FE、BE 节点上。

        1. `hadoop.kerberos.keytab`/`hadoop.kerberos.principal`：用于 Hadoop hdfs 访问，填写 hdfs 对应的值。
        2. `hive.metastore.kerberos.principal`：用于 hive metastore。

    3. 尝试将 principal 中的 ip 换成域名（不要使用默认的 `_HOST` 占位符）
    4. 确认 `/etc/krb5.conf` 文件存在于所有 FE、BE 节点上。
	
3. 访问 HDFS 3.x 时报错：`java.lang.VerifyError: xxx`

   1.2.1 之前的版本中，Doris 依赖的 Hadoop 版本为 2.8。需更新至 2.10.2。或更新 Doris 至 1.2.2 之后的版本。

4. 使用 KMS 访问 HDFS 时报错：`java.security.InvalidKeyException: Illegal key size`

   升级 JDK 版本到 >= Java 8 u162 的版本。或者下载安装 JDK 相应的 JCE Unlimited Strength Jurisdiction Policy Files。

5. 查询 ORC 格式的表，FE 报错 `Could not obtain block`

   对于 ORC 文件，在默认情况下，FE 会访问 HDFS 获取文件信息，进行文件切分。部分情况下，FE 可能无法访问到 HDFS。可以通过添加以下参数解决：

   `"hive.exec.orc.split.strategy" = "BI"`

   其他选项：HYBRID（默认），ETL。