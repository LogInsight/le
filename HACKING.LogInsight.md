1 调试 le 模块 包括本地调试功能, 需要使用 --local 参数启动 le
    参考
    # General domains
    MAIN = 'logentries.com'
    API = 'api.logentries.com'
    DATA = 'data.logentries.com'
    PULL = 'pull.logentries.com'
    # Local debugging
    MAIN_LOCAL = '127.0.0.1'
    API_LOCAL = '127.0.0.1'
    DATA_LOCAL = '127.0.0.1'
    LOCAL = '127.0.0.1'
    
    系统分 四部分, 主站 | API | Data | Pull
 
    其中, pull 是用于通过命令行工具还原 日志文件
    
2 需要先去访问 /agent/account-keys/ ( ACCOUNT_KEYS_API )得到当前用户的 AccountKey
   
   使用 Form 形式进行数据编码
   
   返回一段 json
   
   "accounts" : [
        { "account_key": "xxxxxxxxxxxxxxxx", "name": "xxxxx" },    
   ]
   
   一个 用户 可能有 0 个 或 多个 account , 需要用户选择一个
   
   此时会作为 user_key 保存到文件中
   
   在此处, 修改了 retrieve_account_key , 让流程可以继续
   
3 配置文件的位置, 在普通用户的权限下, 是 ~/.le , 在 root 下是 /etc/le

典型的配置文件如下
 [Main]
user-key = 1234567890111
v1_metrics = False
metrics-mem = system
metrics-token = 
metrics-disk = sum
metrics-swap = system
metrics-space = /
metrics-vcpu = 
metrics-net = sum
metrics-interval = 5s
metrics-cpu = system

4 注册当前 agent 到服务器端
    request = {
        'request': 'register',
        'user_key': config.user_key,
        'name': name,
        'hostname': hostname,
        'system': system,
        'distname': distname,
        'distver': distver
    }
    
    其中, 如果没有在配置文件中 设定了 name, 则 name 为 hostname 的 '.' 之前的字符串
    如果没有 设定 hostname 则, 通过 socket.getfqdn() 得到 网络上的名称
    在 system_detect 中
        distname: distribution name
        distver: distribution version
        kernel: kernel type
        system: system name
        hostname: host name
    
    在里面, 通过 api_request 发送 客户端注册请求 
      (其中 api_request 计算 URL 的算法很奇怪 )
    返回一段 json
    {
        "response": "ok",
        "host": {
          "key": key  // 真正需要的是 key
        }
    }
    
    此数据会保存到配置文件的 agent-key
    
5   follower 关注某个日志文件
    
    在 
   
6   监控全局   
    
    - le 的配置文件可以存放 在服务器端, 服务器端期望返回的数据格式为
      {
        "response": "ok"
        "list": [
            {
                "follow": xxxx
            }
        ]
      }
      在实际世界, 一个被监控的文件需要注意:  
      
    - 默认日志的读取间隔为 0.6s , 并且为写死
    
    