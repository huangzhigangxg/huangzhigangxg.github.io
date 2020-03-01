---
layout: post

title: MoblieScanner 一站式扫描平台[后端]

date: 2019-08-06 15:34:24.000000000 +09:00

---

![1](/assets/mobilescanner/mobilescanner_servers.png)

## Nginx 服务器

### 为什么需要 Nginx？

简单来讲，如果大量的客户端直接与挂在网上的服务器直接链接，每一条链接都是作为一个单独的进程或者线程来处理，随着并发数增加，这台服务早晚会减慢甚至挂掉。

1. 如何让多个服务器可以一起协助完成呢？需要中间有个人分发这些并发的请求，这就是典型的 Nginx负载均衡的能力。
2. 如何让一些重复的api例如一些静态元素，即图片、js或css的资源，避免频繁访问业务服务器，这就是Nginx反向代理的能力，可以缓存这些内容直接给客户端返回结果。

### Nginx 如何配置

```
	1. nginx -t  
	2. vim /home/xiaoju/nginx/conf/nginx.conf
	3. nginx -c /home/xiaoju/nginx/conf/nginx.conf
	4. nginx -s stop
	5. nginx
```
1. 查看配置文件路径 
2. 编辑配置文件
3. 载入配置文件
4. 暂停当前nginx
5. 启动nginx

nginx的配置文件中可以看出几个层级，他们是的作用域是逐级变小的。

1. *全局块*, 全局日志存放等
2. *events块*, 网络连接数等
3. *http块*, 配置代理，缓存等
4. *server块*, 配置虚拟主机的相关参数，监听端口与链接服务应用程序
5. *location块*,请求路由，各个页面的处理情况

```
server {
        listen       80 default_server;
        server_name  _;
        root         /home/xiaoju/msserver/dist; //我们的应用服务入口

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        
		 client_max_body_size 100m;
		 client_body_buffer_size 10m;
		 
        location /api/ {
            proxy_pass http://127.0.0.1:3000/;
        }

        location / {
            try_files $uri $uri/ /index.html;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

### Nignx优势

传统基于进程或线程的模型(Apache就采用这种模型)在处理并发连接时会为每一个连接建立一个单独的进程或线程，且在网络或者输入/输出操作时阻塞。这将导致内存和 CPU 的大量消耗，因为新起一个单独的进程或线程需要准备新的运行时环境，包括堆和栈内存的分配，以及新的执行上下文，当然，这些也会导致多余的 CPU 开销。最终，会由于过多的上下文切换而导致服务器性能变差。
Nginx 的架构设计是采用模块化的、基于事件驱动、异步、单线程且非阻塞，不会新创建新资源，而是复用进程池里的进程，由master 进程负责接受请求，其他进程池里的子进程去争强处理请求任务。所以就省去了创建进程的CPU开销（堆栈分配和上下文切换）。

[参考](https://zhuanlan.zhihu.com/p/74082768)

### 排查问题

1. 查看磁盘 

可能会出现垃圾日志，物理存储紧张，服务器无法连接，程序无法启动

```
df -h 
```

2. 查看端口 

可能会出现因为端口被占用导致nginx重启失败。

```
lsof . 所有端口
lsof -i :80  查看所有接口
```


## Nodejs 

简单说 Node.js 是 JavaScript 运行时环境，能将 JavaScript 转成C/C++ 调用，大大的降低了学习成本。同时 Node.js 也提供了丰富的IO的能力，比如系统api，文件，网络，线程等，内部有libuv支持的事件循环和线程池模拟的异步IO，性能更快。 

![1](/assets/mobilescanner/mobilescanner_nodejs.png)


### Express Web 应用框架

简单说 Express 是基于 Node.js Web 应用程序框架。

1. 可以设置中间件来响应 HTTP 请求 ，统一处理header，错误，IP拦截，日志等。
2. 定义了路由表用于执行不同的 HTTP 请求动作，处理真正的业务能力。
3. 可以通过向模板传递参数来动态渲染 HTML 页面


#### 安装 

NPM 是随同 NodeJS 一起安装的包管理工具，理解为 NodeJS 的代码增强。

```
npm install express --save				  // 框架主题
npm install body-parser --save         // 处理 JSON, Raw, Text 和 URL 编码的数据
npm install cookie-parser --save		  // 解析Cookie的工具
npm install log4js --save				  // 记录日志

```

```
还可以在本地启动 
npm run dev
还可以在服务器启动 
npm run build
```
#### 启动 

```
var http = require('http');
var app = require('../app');		//对应用一系列配置
var server = http.createServer(app);
server.listen(someport); 			//开始监听后程序就完成了初始化，等到请求，并处理，再回调

```

#### 认证如何做？ 

1. Server To User 之间可接入滴滴的登录系统实现采用单点登录，通过ticket和appID每次验证登录状态，其他滴滴同学就可以用自己的账号统一登录了。
2. Server To Server 和滴滴内部codescan平台双方商定保密 req.headers.token。

#### 如何避免回调地狱？

在iOS中也有回调地狱，曾经有过 PromiseKit 来解决，将每一步程序调用包装成 Promise 对象，分离计算(Promise内部实现)与结果(Promise本身这个对象)，将每一步的结果(Promise对象)通过then串联起来，上一个的计算结果作为下一个输入，但如果谋一步发生错误，那么将走catch方法，交给能处理error任务的Promise对象。还有 RxSwift 和 RAC 同样的链式调用，同样也能解决回调地狱的问题，Promise相当于他们的一个子集。

```
- (void)doPromiseAsync {
    [_client loginWithUsername:@"aUser" andPassword:@"123456"].then(^{
        return [_db queryWithUsername:@"aUser"];    
    }).then(^(id userData){
        if (userData) {
            NSNumber * timestamp = [userData timestamp];
            return [_client checkTimestamp:timestamp];
        } else {
            return YES;
        }
    }).then(^(BOOL needFetch){
        if (!needFetch) {
            return;
        }
        return [_client fetchUserData].then(^(id userData){
            return [_db writeUserData:userData];    
        });
    }).then(^{
        NSLog(@"操作成功！");
    }).catch(^(NSError *err){
        NSLog(@"操作失败：%@", err);
    });
}

```
使用 Node.js 8 中 async/await 特性，它是基于 Promise 实现的，它使得异步代码看起来像同步代码。

```
async function test() {
  try {
    toggle_id = "5de483c9db9afc1782263b20";
    toggle = await toggleModelDB.findById(toggle_id);
    apps = await appModelDB.find({ bundle_id: toggle.appidentifier });
    app = apps.pop();
    content = mailContent(toggle.toggles);
    executor = toggle.executor;
    Utils.sendMailForToggle(app, content, executor);
  } catch (err) {
    logger.error(err);
  }
}
```

#### 辅助开发:

##### PM2 node进程管理工具 自动重启

会让自动重启和性能监控，观测日志

```
pm2 list    									//查看进程
pm2 stop all									//杀掉所有进程
pm2 restart all								//重启所有进程
pm2 restart pm2.config.js --env production	//重启所有进程并重置配置文件
```

```
module.exports = {
  apps: [
    {
      name: "msserver",
      script: "./server/bin/www",
      log_date_format: "YYYY-MM-DD HH:mm Z",
      error_file: "../logs/stderr.log",
      out_file: "../logs/stdout.log",
      pid_file: "../pids/child.pid",
      node_args: "--tls-min-v1.0",
      max_restarts: 10, // 最大异常重启次数，即小于min_uptime运行时间重启次数
      autorestart: true, // 默认为true,发生异常的情况下自动重启
      restart_delay: 3000, // 异常重启情况下,延时重启时间
      env: {
        NODE_ENV: "development"
      },
      env_production: {
        NODE_ENV: "production"
      }
    }
  ]
};
```
##### Gitlab CI 自动部署

```
gitlab-ci.yml

stages:
  - test

job1:
  stage: test
  script:
    - cd /home/xiaoju/msserver
    - git fetch --all
    - git reset --hard origin/master
    - npm run build
    - nginx -s stop
    - nginx
    - pm2 restart pm2.config.js --env production
  only:
    - master

```
##### 日志检索

```
head -n 1000 stderr.log 查看前面1000行

tail -n 1000 stderr.log 查看最后1000行

grep '2019-11-04T21:54:50.876' server-2019-11-04.log

grep  -C 20 '5ddf4b4445daf8a23f52ade5' server-2019-11-28.log 前后20行

cat filename | head -n 3000 | tail -n +1000 查看1000到3000行的数据

scp root@XX.XX.XX.X:/home/xiaoju/logs/server-2019-11-28.log /Users/xiaogang/Desktop/server-2019-11-28.log 

```
#### Nodejs 的优势

##### 直接js翻译到机器码

V8采用即时编译技术（JIT），直接将JavaScript代码编译成本地平台的机器码。宏观上看，其步骤为JavaScript源码—>抽象语法树—>本地机器码，并且后一个步骤只依赖前一个步骤。这与其他解释器不同，例如Java语言需要先将源码编译成字节码，然后给JVM解释执行，JVM根据优化策略，运行过程中有选择地将一部分字节码编译成本地机器码。V8不生成中间代码，一步到位，编译成机器码，CPU就开始执行了。比起生成中间码解释执行的方式，V8的策略省去了一个步骤，程序会更早地开始运行。并且执行编译好的机器指令，也比解释执行中间码的速度更快。不足的是，缺少字节码这个中间表示，使得代码优化变得更困难。
与传统的「编译-解析-执行」的流程不同，V8 处理 JavaScript，并没有二进制码或其他的中间码。
简单来说，V8主要工作就是：「把 JavaScript 直译成机器码，然后运行」


##### 单线程提交，多线程执行，包装了多线程任务，任何人都能写出高性能的代码
一个CPU只能处理一个线程，nodejs中的libuv 提供了少量线程数，线程池里可以复用这些线程。减少线程之间的切换。外面js层单一线程不断提交请求，后面libuv是多线程在处理，并处理完成后通知。

[狼叔：如何正确的学习Node.js](https://i5ting.github.io/How-to-learn-node-correctly/)

[深入浅出 Node.js（五）：初探 Node.js 的异步 I/O 实现](https://www.infoq.cn/article/nodejs-asynchronous-io)


## Mongodb数据库

### mongoose 

Mongoose是在node.js环境下对mongodb进行便捷操作的对象模型工具，需要安装node.js环境以及mongodb数据库。
 
### 安装

npm install mongoose --save

### 链接

```
var mongoose = require("mongoose");
var DB_URL = "mongodb://localhost:27017/mobilescanner";
var CONNECT_OPTION = {
  useNewUrlParser: true
};

mongoose.connect(DB_URL, CONNECT_OPTION)
```

### 启动

mongod --config /usr/local/etc/mongod.conf

```
systemLog:
  destination: file  #日志输出方式
  path: /usr/local/var/log/mongodb/mongo.log #日志路径
  logAppend: true #启动时，日志追加在已有日志文件内还是备份旧日志后，创建新文件记录日志
storage:
  dbPath: /usr/local/var/mongodb  #数据库，默认/data/db
net:
  bindIp: 127.0.0.1	#绑定监听的ip

```

[中文参考](http://www.mongoosejs.net/docs/connections.html)

[英文参考](https://docs.mongodb.com/manual/reference/operator/aggregation/match/)

## 技术规划方向

目前 MobileScanner 还只是企业内部应用的单机架构，用户量少对大吞吐并行能力要求不高。
这只是后台开发的一次有应用场景的实践，真正线上服务还会面连各种各样的问题

1. 业务主机和数据库主机分离，避免两者抢占计算资源
2. 在业务主机引入Redis本地缓存，提前拦截到大量服务器请求
3. 引入反向代理实现负载均衡，多个业务主机一起计算
4. 数据库读写分离，提高读数据库的效率
5. 数据库按业务分库，分流数据库请求
6. 数据库中的大表拆成小表，提高数据检索效率
7. 使用LVS或F5来使多个Nginx负载均衡，从TCP层接做负载均衡，效率更高
8. 通过DNS轮询实现机房间的负载均衡
9. 等等

以后的对后台技术还要不断尝试，了解大家正在解决的问题。

[分布式架构演进过程](https://zhuanlan.zhihu.com/p/67541032)
