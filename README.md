# osmonitor-6block
在[osmonitor](https://github.com/bitxx/osmonitor) 的基础上，临时改造的一个6block版本。用于方便非专业的运维人员能及时知道哪台设备异常，并及时处理

![start](/logo.png)

## 针对6block的功能
1. 由客户端(client)和服务端(server)两部分组成，客户端在各个worker机上，用来上报异常，服务端用来接收异常并发送邮件
2. 服务端负责接收各台worker机器的异常上报，若接收到异常上报，则定时发送邮件给用户。邮件发送时间由用户自行设置，我一般30分钟接收一次
3. 主要上报异常说明
   1. worker进程监控，因各种原因崩溃或者丢失，都会发送给服务端，然后邮件告诉你，有worker机的程序崩了
   2. 主要是监控worker进程没崩，但算力归零时的情况，也就是有卡显示算力为`N/A`的问题。我主要是读取`journalctl`最近1分钟的日志，每分钟读一次，连续读5分钟，如果都出现`N/A`标记，则上报异常给服务端
4. `需要再次解释`，服务端邮箱不是收到异常上报就立马发送邮件，而是根据你设置的间隔时间，统一发送。先汇总，后发送 

## 针对6block的使用
功能介绍、编译等细节请参考[osmonitor](https://github.com/bitxx/osmonitor) 里面的介绍，这里只针对6block的使用进行`最简步骤说明`
这里直接从[release](https://github.com/bitxx/osmonitor-6block/releases)下载程序进行操作，我只编译了linux-amd64的。不放心的或者想用别的平台，可以自行查阅代码并编译，本开源项目不对任何风险负责。  

## 注意
worker设备，必须使用service方式部署，也就是要支持`systemctl`启动停止锄头，`journalctl`查阅程序日志

### client 客户端
```shell
# 命令行方式启动
## --name worker名称，每台机子的名称，每台不能设置一样。方便邮件提醒时，区分是哪台
## --secret 和服务器端交互使用的私钥，这个服务端来配置
## --server-url 服务端地址，服务端的地址
## --proc-names 一定要写，你在systemctl上用的什么名字控制锄头，这里就写成什么名字，不要写错，要不然就没法正常监控了

## 命令示例
./client start --name test-client --secret 123456 --server-url ws://192.168.1.2:8888 --proc-names 6block-miner
```

### server 服务端
```shell
# 命令行方式启动
## --name 服务端名称，随意起个名字
## --secret 和客户端交互使用的私钥，用于身份鉴定，客户端和这里需要保持一致，请改个更复杂点的，这里我用了123456做例子
## --host 服务端地址
## --port 端口
## --email-subject-prefix 邮件标题前缀
## --email-host 邮箱服务地址，如：smtp.office365.com
## --email-port 邮箱端口，如：587
## --email-username 发件邮箱账户 
## --email-password 邮箱密钥 
## --email-from 发件邮箱账户 
## --email-to 收件邮箱账户(多个逗号隔开) 
## --email-monitor-time 邮件发送间隔，单位秒,也就是你汇总到异常信息后，多久发送一次邮件，我这里设置半小时，1800秒，如果想要更勤快点监控，可以设置更短

## 命令示例，邮箱以outlook为例
./server start --name 自己随意起个名字 --secret 123456 --host 0.0.0.0 --port 8003 --email-subject-prefix aleo --email-host smtp.office365.com --email-port 587 --email-username xxx@outlook.com --email-password 邮箱密码 --email-from xxx@outlook.com --email-to xxx@outlook.com --email-monitor-time 1800
```
