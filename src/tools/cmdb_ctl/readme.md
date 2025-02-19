# cmdb 小工具

### 动态调整日志级别
- 使用方式

  ```
  ./tool_ctl log [flags]
  ```

- 命令行参数
  ```
  --set-default      [=false]: set log level to default value
  --set-v             =""    : set log level for V logs
  --addrport          =""    : the ip address and port for the hosts to apply command, separated by comma
  --zk-addr           =""    : the ip address and port for the zookeeper hosts, separated by comma, corresponding environment variable is ZK_ADDR
  --redis-addr        =""    : assign redis server address default is 127.0.0.1:6379
  --redis-pwd         =""    : assign redis server password
  --redis-database    =""    : assign the redis database default is 0
  --redis-mastername  =""    : assign redis server master name defalut is null
  --redis-sentinelpwd =""    : assign the redis sentinel password  default is null
  ```
- 示例

  - ```
    ./tool_ctl log --set-default --addrport=127.0.0.1:8080 --zk-addr=127.0.0.1:2181
    ```

  - ```
    ./tool_ctl log --set-v=3 --addrport="127.0.0.1:8080" --zk-addr="127.0.0.1:2181"
    ```

### 优雅退出
- 使用方式

  ```
  ./tool_ctl shutdown [flags]
  ```

- 命令行参数
  ```
  --pids="": the processes to be shutdown, separated by comma
  --show-pids[=false]: show cmdb processes pid
  ```
- 示例

  - ```
    ./tool_ctl shutdown --pids=1234
    ```

  - ```
    ./tool_ctl shutdown --show-pids
    ```

### zookeeper操作
- 使用方式

  ```
  ./tool_ctl zk [command]
  ```

- 子命令
  ```
  ls          list children of specified zookeeper node
  get         get value of specified zookeeper node
  del         delete specified zookeeper node
  set         set value of specified zookeeper node
  ```
- 命令行参数
  ```
  --zk-path="": the zookeeper path
  --zk-addr="": the ip address and port for the zookeeper hosts, separated by comma, corresponding environment variable is ZK_ADDR
  --value="": the value to be set（仅用于set命令）
  ```
- 示例

  - ```
    ./tool_ctl zk ls --zk-path=/test --zk-addr=127.0.0.1:2181
    ```

  - ```
    ./tool_ctl zk get --zk-path=/test --zk-addr=127.0.0.1:2181
    ```

  - ```
    ./tool_ctl zk del --zk-path=/test --zk-addr=127.0.0.1:2181
    ```

  - ```
    ./tool_ctl zk set --zk-path=/test --zk-addr=127.0.0.1:2181 --value=test
    ```
    
### 回显服务器
- 使用方式

  ```
  ./tool_ctl echo [flags]
  ```

- 命令行参数
  ```
  --url="": the url for echo server to listen
  ```
- 示例

  - ```
    ./tool_ctl echo --url=127.0.0.1:8080/echo
    ```
### 检查主机快照
- 使用方式

  ```
  ./tool_ctl snapshot [flags]
  ```

- 命令行参数
  ```
  --bizId=2: blueking business id. e.g: 2
  --zk-addr="": the ip address and port for the zookeeper hosts, separated by comma, corresponding environment variable is ZK_ADDR
  ```
- 示例

  - ```
    ./tool_ctl snapshot --bizId=2 --zk-addr=127.0.0.1:2181
    ```

 ### 权限中心资源操作
 - 使用方式
 
   ```
   ./tool_ctl auth [command]
   ```

 - 子命令
   ```
   check       check if user has the authority to operate resources
   ```

 - 命令行参数
  ```
   -h, --help[=false]: help for auth
   -v, --logV=4: the log level of request, default, request body log level is 4
   -r, --resource="": the resource for authorize
   -f, --rsc-file="": the resource file path for authorize
  --zk-addr="": the ip address and port for the zookeeper hosts, separated by comma, corresponding environment variable is ZK_ADDR
   --supplier-account="0": the supplier id that this user belongs to（仅用于check命令）
   --user="": the name of the user（仅用于check命令）
   ```

 - 示例
 
   - ```
     ./tool_ctl auth check --app-code=test --app-secret=test --auth-address=http://127.0.0.1 --resource=[{\"type\":\"modelInstance\",\"action\":\"update\",\"Name\":\"\",\"InstanceID\":1,\"InstanceIDEx\":\"\",\"SupplierAccount\":\"\",\"business_id\":2,\"Layers\":[{\"type\":\"model\",\"action\":\"\",\"Name\":\"\",\"InstanceID\":1,\"InstanceIDEx\":\"\"}]}] --supplier-account=0 --user=test
     ```

   - ```
     ./tool_ctl auth check --app-code=test --app-secret=test --auth-address=http://127.0.0.1 --rsc-file=resource.json
     
     resource.json file content example:
     [
         {
             "type": "modelInstance",
             "action": "update",
             "Name": "",
             "InstanceID": 1,
             "InstanceIDEx": "",
             "SupplierAccount": "",
             "business_id": 2,
             "Layers": [
                 {
                     "type": "model",
                     "action": "",
                     "Name": "",
                     "InstanceID": 1,
                     "InstanceIDEx": ""
                 }
             ]
         }
     ]
     ```

### 检查拓扑结构
- 使用方式

  ```
  ./tool_ctl topo [flags]
  ```

- 命令行参数
  ```
  --bizId=2: blueking business id. e.g: 2
  --mongo-uri="": the mongodb URI, eg. mongodb://127.0.0.1:27017/cmdb, corresponding environment variable is MONGO_URI
  ```
- 示例

  - ```
    ./tool_ctl topo --bizId=2 --mongo-uri=mongodb://127.0.0.1:27017/cmdb
    ```

### 操作api请求限流策略
- 使用方式
    ```
      ./tool_ctl limiter [flags]
      ./tool_ctl limiter [command]
    ```

- 子命令
    ```
      set         set api limiter rule, use with flag --rule
      get         get api limiter rules according rule names,use with flag --rulenames
      del         del api limiter rules, use with flag --rulenames
      ls          list all api limiter rules
    ```
- 命令行参数
    ```
     --rule="": the api limiter rule to set, a json like '{"rulename":"rule1","appcode":"gse","user":"","ip":"","method":"POST","url":"^/api/v3/module/search/[^\\s/]+/[0-9]+/[0-9]+/?$","limit":1000,"ttl":60,"denyall":false}'
     --rulenames="": the api limiter rule names to get or del, multiple names is separated with ',',like 'name1,name2'
    ```

- rule策略字段说明

| 字段     | 类型   | 必选 | 描述                                                         |
|----------|--------|------|--------------------------------------------------------------|
| rulename | string | 是   | 策略名                                                       |
| appcode  | string | 否   | 应用ID                                                       |
| user     | string | 否   | 请求发起的用户名                                             |
| ip       | string | 否   | api的来源ip                                                  |
| method   | string | 否   | 请求的类型，配置的情况下只能为POST、GET、PUT、DELETE中的一种 |
| url      | string | 否   | api的url正则表达式                                           |
| limit    | int64  | 否   | api请求限制总次数                                            |
| ttl      | int64  | 否   | 策略存活时间，单位为秒                                       |
| denyall  | bool   | 否   | 是否直接禁掉请求，默认为false，为true时忽略limit和ttl参数    |
 
appcode、user、ip、method、url需要至少配置一项  
denyall配置为false的情况下，limit和ttl配置才能生效

- 示例
    ```
      以下命令是在配置了ZK_ADDR环境变量的情况下使用，没有配置时也可以通过命令行参数--zk-addr指定
      # 列出所有策略
      ./tool_ctl limiter ls
      # 配置策略，对url限制请求次数
      ./tool_ctl limiter set --rule='{"rulename":"rule1","appcode":"gse","user":"admin","ip":"","method":"POST","url":"^/api/v3/module/search/[^\\s/]+/[0-9]+/[0-9]+/?$","limit":1000,"ttl":60,"denyall":false}'
      # 配置策略，将url直接禁掉
      ./tool_ctl limiter set --rule='{"rulename":"rule1","appcode":"gse","user":"admin","url":"^/api/v3/module/search/[^\\s/]+/[0-9]+/[0-9]+/?$","denyall":true}'
      # 获取某些策略详情
      ./tool_ctl limiter get --rulenames=test1,test2
      # 删除某些策略
      ./tool_ctl limiter del --rulenames=test1,test2
    ```
### 检查yaml格式文件是否正确
- 使用方式
    ```
      ./tool_ctl checkconf [flags]
    ```
- 命令行参数
    ```
      --dir="": the directory path where the configuration file is located
      --file="": the path of the configuration file
    ```
- 示例
    ```
      ./tool_ctl checkconf --dir="/data/cmdb/cmdb_adminserver/configures"
      ./tool_ctl checkconf --file="/data/cmdb/cmdb_adminserver/configures/common.yaml"
    ```
### DB操作(目前支持查找、删除、显示collections操作)

- 使用方式
     ```
         ./tool_ctl db [flags]
         ./tool_ctl db [command]
     ```
- 子命令
     ```
          find        find eligible data from the db
          delete      delete eligible data from the db
          show        show all collections
     ```

- 命令行参数
     ```
          --collection=""   : collection name
          --condition=""    : query conditions
          --num=5           : numbers of result to show                  （默认值是5，仅限find命令）
          --pretty[=false]  : query result are displayed in JSON format  （默认按照文本显示，仅限find命令）
          --resfilter=""    : display the required fields                （仅限find命令）
     ```
- 示例
     ```
         对DB进行查找操作示例:
             ./tool_ctl --mongo-uri="mongodb://user:pwd@localhost:27017/test?replicaSet=rs0" db find --collection="test_collection" --condition="{\"bk_biz_id\" : 6}"  --resfilter="id,type" --pretty=true
         回显样式:
             [
             	{
             		"current_version": "y3.X.XXXXXXXX",
             		"init_distro_version": "XXXX.XXXX.XXXX"
             	}
             ]
             total data num is 1
     ```
     ```
         显示指定DB中的所有collections
             ./tool_ctl --mongo-uri="mongodb://user:pwd@localhost:27017/test?replicaSet=rs0" db show
         回显样式:
             cc_test1
             cc_test2
             cc_test3
             cc_test4
             cc_test5
             total collection num is 5
     ```
     ```
          对DB进行删除操作示例:
              ./tool_ctl --mongo-uri="mongodb://user:pwd@localhost:27017/test?replicaSet=rs0" db delete --collection="test_collection" --condition="{\"bk_biz_id\" : 6}"
          回显样式
              delete total data num is 3
     ```
     
### Redis操作(目前支持scan、scan-del操作)
- 使用方式
     ```
         ./tool_ctl redis [flags]
         ./tool_ctl redis [command]
     ```
- 子命令
     ```
          scan            scan the redis
          scan-del        del scanned keys
     ```     
     
- 命令行参数
     ```
          match            redis scan  match pattern  default is null
     ```
- 示例
     ```
         对Redis进行 scan 操作示例:
             ./tool_ctl redis --redis-pwd="admin" scan --match="test*" 
         回显样式:
             show some results as an example :
             test2
             test1
             test
             example end
          命令说明：
             此处只显示不多于5个结果示例，并不是全部结果
                
          对Redis进行 scan-del 操作示例:
             ./tool_ctl redis --redis-pwd="123456" scan-del --match="test*"
          回显样式:  
             del keys start :
             del keys num: 1.
             del keys num: 2.
             del keys success ,total num is 3
          命令说明：
             按照指定的match参数进行匹配，会一次性将匹配到的key进行全部删除，此命令需要慎用！
     ```