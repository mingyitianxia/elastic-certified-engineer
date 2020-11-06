## INSTALLATION AND CONFIGURATION

集群安装和配置

## GOAL: Secure a cluster and an index using Elasticsearch Security

目标：用ES的安全模块保护集群和索引

## REQUIRED SETUP

### 第0题，按要求创建集群

1. a running Elasticsearch cluster with at least one node and a Kibana instance
   1. 创建一个最少拥有1个ES节点1个Kibana节点的集群
1. no index with name `hamlet` is indexed on the cluster
   1. 在集群中没有叫`hamlet`的索引

### 第0题，题解

1. 初始化集群
   1. rpm/命令 方式
      1. `sh $ES_HOME/bin/elasticsearch -d`
      1. `sh $KIBANA_HOME/bin kibana`
      1. `systemctl start elasticsearch.server`
      1. `systemctl start kibana.server`
   1. docker 方式
      1. `docker run -p 9200:9200 -p 9300:9300 --name elasticsearch elasticsearch:7.2.1 -d`
      1. `docker-compose -f 1e1k_base_cluster.yml up -d --build`
   1. 参考链接和页面路径不贴了，根据前人的经验，这题可能会出现某些节点没启动，配合`elasticsearch.yml`等配置文件的检查做就好。
1. 清理索引
   1. 在kibana里运行`DELETE hamlet`
   1. 在命令行里运行`curl -X DELETE http://localhost:9200/hamlet`
   1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-delete-index.html)
   1. 页面路径：Indices APIS =》 Delete index

### 第0题，题解说明

* 这题没什么特别需要说明的，创建和启动集群前一章已经讲过了，这里主要需要注意的是kibana的配置和进程
  1. 配置：
     1. kibana的配置中需要添加`elasticsearch.hosts: ["http://localhost:9200",]`。由于kibana是一个前端项目，所以这里需要一个完整的可以被前端访问的url才行。
     2. 当ES开启了认证之后，kibana中需要添加针对`kibana`这个账户的账号密码（通过ES自带工具生成或指定），以保证kibana实体可以连接到ES节点，但是在登录kibana页面的时候需要填写的是`elastic`这个账户的账号密码。
  2. 进程：
     1. kibana是通过nodejs启动的，所以通过命令`ps -ef | grep kibana`是找不到他的进程的，正确的做法是通过`lsof -i:5601`找到监听5601（或者其他指定的kibana监听端口的进程），然后通过进程 pid 来找。
     2. 7.x 的kibana的进程多半会长得类似`./../node/bin/node ./../src/cli`

### 第1题，配置集群安全

1. Enable xPack security on the cluster
   1. 开启x-pack保护集群
1. Set the password of the `elastic` and `kibana` built-in users. Use the pattern "{{username}}-password" (e.g., "elastic-password")
   1. 以"{{username}}-password"为模板，给`elastic`和`kibana`等内置账户设置密码
1. Login to Kibana using the `elastic` user credentials
   1. 用`elastic`的认证信息登录 kibana

### 第1题，题解

1. 修改配置文件`$ES_HOME/config/elasticsearch.yml`
   1. 添加`xpack.security.enabled: true`
   2. 有的时候可能启动报错，还需要添加配套的`xpack.security.transport.ssl.enabled=true`
2. 在ES安装目录里（包含bin目录的那个）运行`bin/elasticsearch-setup-passwords interactive`给用户设置密码
   1. docker 运行的同学先要通过`docker exec -it elasticsearch bash`登录到docker实例里面，然后再执行
   2. 物理机部署的同学直接进入`$ES_HOME`就好

### 第1题，题解说明

* 这题分两个部分，开启验证和设置密码
  1. 开启认证是需要修改ES的配置文件才能生效的，所以会涉及到`$ES_HOME/config/elasticsearch.yml`
     1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-settings.html#general-security-settings)
     1. 页面路径：Set up Elasticsearch =》 Configuring Elasticsearch =》Security settings
  2. 第一次设置密码得通过ES命令行工具进行设置，后面的创建用户、修改密码等可以通过kibana的[ Management > Users]页面来配置。
     1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/setup-passwords.html)
     2. 页面路径：Command line tools =》 elasticsearch-setup-passwords

### 第2题，添加数据

We are now going to use the _bulk API to index some documents into the cluster. The documents are lines from Hamlet by William Shakespeare, and have the following structure:
   1. 来让我们通过bulk API把莎士比亚写的一些哈姆雷特第句子存到ES里，数据结构如下
```json
{
  "line_number": "String",
  "speaker": "String",
  "text_entry": "String",
}
```

1. Let’s continue with the exercise.
   1. Create the index `hamlet` and add some documents by running the following _bulk command:
   2. 通过下面一些bulk的命令把哈姆雷特的句子存进ES（这些命令基本上得在kibana里执行，因为他们不是正常的curl命令）
```bash
PUT hamlet/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"5","speaker":"BERNARDO","text_entry":"He."}
{"index":{"_index":"hamlet","_id":5}}
{"line_number":"6","speaker":"FRANCISCO","text_entry":"You come most carefully upon your hour."}
{"index":{"_index":"hamlet","_id":6}}
{"line_number":"7","speaker":"BERNARDO","text_entry":"Tis now struck twelve; get thee to bed, Francisco."}
{"index":{"_index":"hamlet","_id":7}}
{"line_number":"8","speaker":"FRANCISCO","text_entry":"For this relief much thanks: tis bitter cold,"}
{"index":{"_index":"hamlet","_id":8}}
{"line_number":"9","speaker":"FRANCISCO","text_entry":"And I am sick at heart."}
{"index":{"_index":"hamlet","_id":9}}
{"line_number":"10","speaker":"BERNARDO","text_entry":"Have you had quiet guard?"}
```

### 第2题，题解

1. 添加数据的时候已经指定了数据结构，如果直接通过下面的bulk命令进行插入的话，索引也可以建立成功，因为ES会根据数据结构里字段的种类尝试进行索引，但是无论是考试还是平时工作的时候，个人都建议先创建索引，这样ES就不需要再分析数据结构尝试进行映射了。

```bash
PUT hamlet
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "line_number": {
        "type": "text"
      },
      "speaker": {
        "type": "text"
      },
      "text_entry": {
        "type": "text"
      }
    }
  }
}
```

1. 然后在kibana的dev-tool里执行上面那些命令，贴在一起直接可以一起执行。

### 第2题，题解说明

* 这里只是准备数据的部分，但是也涉及了index data的部分
  1. 索引的构建，因为指定了数据结构了，所以最好还是预先创建索引，后面可能还会有更多的修改重建的过程
     1. 这里需要注意的是，ES的7.x里已经不能指定type了，统一使用"_doc"，所以mapping里面不需要指定type的名字，直接就是properties了。
     2. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-create-index.html)
     3. 页面路径：Indices APIs =》 Create Index
  2. 通过bulk API插入数据，没什么特别介绍，注意数据是第一行写索引信息和索引命令，第二行写数据就好
     1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-bulk.html)
     2. 页面路径：Document APIs =》 Bulk API

### 第3题，权限认证

You can specify authentication (“who are you”) and authorisation (“what you can do”) policies on the Elasticsearch resources by means of users, roles, and mappings between users and roles. Do you know how to do that?
  1. 通过users、roles，你可以在ES里指定用户、权限以及给特定用户赋予某些权限。

1. Create the security role `francisco_role` in the native realm, so that:
   1. 创建一个叫`francisco_role`的安全组，它包含了：
   1. the role has "monitor" privileges on the cluster,
      1. 它拥有"monitor"集群的权限
   1. the role has all privileges on the `hamlet` index
      1. 这个role对索引`hamlet`拥有全部权限
1. Create the user `francisco` with password "francisco-password"
   1. 创建一个用户，账户密码分别是 `francisco` 和 `francisco-password`
2. Assign the role `francisco_role` to the `francisco` user
   1. 给`francisco`赋予`francisco_role`的权限组
3. Login using the `francisco` user credentials, and run queries on `hamlet` to verify that the role privileges were correctly set
   1. 用`francisco`登录系统，然后对`hamlet`执行一些查询以确保权限被设置正确了。

```bash
GET hamlet/_search
PUT hamlet/_doc/1
{"tesst":"1111"}
DELETE hamlet
GET _cat/indices
DELETE .kibana
```

### 第3题，题解

1. 用`elastic`登录kibana
2. 点左侧齿轮 ⚙️ （manage）
3. 在Roles的菜单里点`Create role`
   1. Elasticsearch 栏目：
      1. __Role name__ 填 `francisco_role`
      2. __Cluster privileges__ 选 `monitor`
      3. __Run As privileges__ 空着
      4. __Index privileges__
         1. __Indices__ 写 `hamlet`
         2. __Privileges__ 选 `all`
   2. Kibana 栏目：
      1. 点 `Add space privilege`
      2. __Space__ 选 `Default`
      3. __Privileges__ 选 `all`
4. 在Users的菜单里点`Create user`
   1. 账户密码分别填 `francisco` 和 `francisco-password`
   2. __Roles__ 选 `francisco_role`
5. 点右上角头像，弹框里点 `Log out`
6. 登录用`francisco` 和 `francisco-password`
7. 运行上面 ⬆️ 那些命令
   1. 前几个都直接返回成功
   2. `DELETE .kibana`会返回下面 ⬇️ 报错，意思是当前用户对`.kibana`没有权限执行这个命令
    ```json
      {
        "error": {
          "root_cause": [
            {
              "type": "illegal_argument_exception",
              "reason": "The provided expression [.kibana] matches an alias, specify the corresponding concrete indices instead."
            }
          ],
          "type": "illegal_argument_exception",
          "reason": "The provided expression [.kibana] matches an alias, specify the corresponding concrete indices instead."
        },
        "status": 400
      }
    ```

### 第3题，题解说明

* 这题主要考察用户权限以及kibana的操作
  1. 这里比较可能坑的是创建role的时候要选择kibana的space，否则即使账号创建好了，权限也设置好了，依然无法登录，报
   ```json
   {
    "statusCode": 403,
    "message": "Forbidden",
    "error": "Forbidden"
   }
   ```
  1. [参考连接-ES-权限说明](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-privileges.html#privileges-list-indices)
  2. [参考链接-ES-UserAPI](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-user.html)
  3. [参考链接-ES-RoleAPI](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-role.html)
  5. [参考链接-Kibana](https://www.elastic.co/guide/en/kibana/7.2/secure-reporting.html)

### 第4题，通过命令来完成用户/权限配置
这个题需要纯dev-tool执行，个人觉得不太会考，但是现在认证工程师越来越多，不排除官方为了提高门槛而把它拿来当考题的可能。

Not bad, right? Now, let’s create a more sophisticated security role, which assigns read-only permissions on indices, documents and fields.

并不难对吧，让我们弄个更复杂的安全组，指定`read-only`的权限在索引、文档、字段上

1. Create the security role `bernardo_role` in the native realm, so that:
   1. 创建一个叫`bernardo_role`的安全组
   2. the role has "monitor" privileges on the cluster,
      1. 它拥有"monitor"集群的权限
   3. the role has read-only privileges on the `hamlet` index,
      1. 它对`hamlet`这索引有只读权限
   4. the role can see only reach (reach 原文里没有， 为了意义完整加在这里）those documents having "BERNARDO" as a `speaker`,
      1. 它只能访问那些 `speaker`字段是"BERNARDO"的文档
   5. the role can see only the `text_entry` field
      1. 它只能看到`text_entry`这一个字段
2. Create the user `bernardo` with password "bernardo-password"
   1. 创建个新账号`bernardo`，设置密码为"bernardo-password"
3. Assign the role `bernardo_role` to the `bernardo` user
   1. 给账号`bernardo`指定`bernardo_role`安全组

### 第4题，题解

1. 在kibana里输入以下命令来创建role
```bash
POST /_security/role/bernardo_role
{
  "run_as": [ "bernardo" ],
  "cluster": [ "monitor" ],
  "indices": [
    {
      "names": [ "hamlet" ],
      "privileges": [ "read" ],
      "field_security" : {
         "grant" : [ "text_entry" ]
      },
       "query": "{\"match\": {\"speaker\": \"BERNARDO\"}}"
    }
  ]
}
```

1. 再输入以下命令创建用户
```bash
POST /_security/user/bernardo
{
  "password" : "bernardo-password",
  "roles" : [ "bernardo_role"],
  "full_name" : "Jack Nicholson",
  "email" : "test@example.com"
}
```

### 第4题，题解说明

* 创建账户、安全组都可以通过接口完成
   1. 创建安全组：
      1. [参考连接-创建](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-role.html)
         1. 页面路径：X-Pack APIs =》 Security APIs =》 Create or update roles
      2. [参考链接-设置安全组](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/defining-roles.html)
         1. 页面路径：Secure a cluster =》 User authorization =》 Defining roles
      3. [参考链接-query和可见字段的部分](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/field-and-document-access-control.html)
         1. 页面路径：Secure a cluster =》 User authorization =》 Setting up field and document level security
   2. 创建用户：
      1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-user.html)
         1. 页面路径：X-Pack APIs =》 Security APIs =》 Create or update users
* 这里有几个比较坑爹的地方
  1. 在这题中的一些设置需要开通xpack的收费项目，一个解决办法是在kibana里试用一下，一个月时间用起来，另一个就只有答案背下来了。
     1. 如果没开相关权限的话，可能role的设置会提示你当前license没有 __字段/文档级别权限设置__ 的权力。role里的`query`和`field_security`都属于 __字段/文档级别权限设置__
      ```json
      {
        "error": {
          "root_cause": [
            {
              "type": "security_exception",
              "reason": "current license is non-compliant for [field and document level security]",
              "license.expired.feature": "field and document level security"
            }
          ],
          "type": "security_exception",
          "reason": "current license is non-compliant for [field and document level security]",
          "license.expired.feature": "field and document level security"
        },
        "status": 403
      }
      ```

  2. 在role里面的field可见性在role的设置的章节里没有，要转跳两次在【[这个连接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/field-level-security.html)】里才提到具体应该怎么设
  3. role里的数据筛选的query需要对原来的json做jsonToString的操作，因为这个字段是个String型的数据。
* 所以即使单独使用下面这俩设置，都有可能提示权限不够。
    ```bash
    POST /_security/role/bernardo_role
    {
      "indices": [
        {
          "names": [ "*" ],
          "privileges": [ "read" ],
          "query": "{\"match\": {\"speaker\": \"BERNARDO\"}}"
        }
      ]
    }
    ```

    ```bash
    POST /_security/role/bernardo_role
    {
      "indices": [
        {
          "names": [ "hamlet" ],
          "privileges": [ "read" ],
          "field_security" : {
            "grant" : [ "text_entry" ]
          }
        }
      ]
    }
    ```

### 第5题，改密码

1. Login using the `bernardo` user credentials, and run queries on  `hamlet` to verify that the role privileges were correctly set.
   1. 用`bernardo`的账户密码登录，然后对`hamlet`这个索引进行一些query操作来验证一下我们的权限设置对了没
2. Whoops, I asked you to assign the wrong password to the “bernardo” user. My bad. Would you be so kind as to change it?
   1. oops，我让你给`bernardo`设错密码了，你能帮我改一下吗？


1. Change the password of the `bernardo` user to "poor-bernardo"
(Never forget to check if it worked!)
   1. 给`bernardo` 的密码改成 "poor-bernardo"，（别忘了试一下密码对了没）

### 第5题，题解

1. 在kibana里输入以下命令
```bash
POST /_security/user/bernardo/_password
{
  "password": "poor-bernardo"
}
```

### 第5题，题解说明
* 这题就很简单的改密码，但是有个注意点是url里面的用户名（这里是`bernardo`），如果不指定就改的是当前登录的用户的密码，为了避免改错用户，还是建议在这里明确指定好。
  1. 这里的用户操作也会受权限所限制，比如用一个受限用户进行改密码操作，可能会提示权限不足。
   ```json
    {
      "error": {
        "root_cause": [
          {
            "type": "security_exception",
            "reason": "action [cluster:admin/xpack/security/user/change_password] is unauthorized for user [test123]"
          }
        ],
        "type": "security_exception",
        "reason": "action [cluster:admin/xpack/security/user/change_password] is unauthorized for user [test123]"
      },
      "status": 403
    }
   ```
   1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-change-password.html)
      1. 页面路径：X-Pack APIs =》 Security APIs =》 Change passwords
