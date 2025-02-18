# YTTech intern

## 2.10 环境配置

[后台入职流程](https://wiki-back.aicoin.com/workspace/c0685860-91a7-4f37-983b-d86d2f43dec1/1FOP4BihD65-G0yqpQ_26?mode=page)

### apple

总是显示目前无法创建账户，后面用浏览器在官网注册成功。  


### Golang

Go 1.18版本因为上游不再支持而被Homebrew禁用了。这意味着通过常规的Homebrew命令直接安装这个特定版本的Go已经不可行。
可以使用**goenv**，这是一个非常流行的Go版本管理工具，允许你安装和切换不同版本的Go。

- 安装`goenv`：

    ```bash
    brew install goenv
    ```

- 设置环境变量（根据你的shell，可能是`.bash_profile`, `.zshrc`, 等）：

    ```bash
    echo 'export PATH="$HOME/.goenv/bin:$PATH"' >> ~/.zshrc
    echo 'eval "$(goenv init -)"' >> ~/.zshrc
    source ~/.zshrc
    ```

- 安装你需要的Go版本：

    ```bash
    goenv install 1.18
    ```

- 切换到该版本：

    ```bash
    goenv global 1.18
    ```

### cursor

从官网下载的，下载了好久，挂了我自己的梯子也没用。
后面龙哥帮我开了代理，用`Proxy SwitchyOmega` 插件，下载成功了。

### 快捷键

使用快捷键 Ctrl + ` （反引号，通常位于键盘左上角，与波浪号“~”共享按键）。这个快捷键是默认设置用来快速切换和打开集成终端的。

### PHP

[brew安装](
https://blog.csdn.net/chenpy/article/details/142646238?ops_request_misc=%257B%2522request%255Fid%2522%253A%252273a3df64b639922815862634957162ab%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=73a3df64b639922815862634957162ab&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-142646238-null-null.142^v101^pc_search_result_base7&utm_term=macbook%E5%AE%89%E8%A3%85brew&spm=1018.2226.3001.4187)

```bash
# 镜像：
git config --global url."https://gitclone.com/github.com/".insteadOf "https://github.com/"

# 手动克隆仓库:
git clone https://github.com/shivammathur/homebrew-php /usr/local/Homebrew/Library/Taps/shivammathur/homebrew-php

brew tap shivammathur/php
brew search php
brew install shivammathur/php/php@7.3

```

### Gitlab

[双重认证](https://blog.csdn.net/xh365647/article/details/135848520?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522c6428d506cac3eb6ad7e319c91099318%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=c6428d506cac3eb6ad7e319c91099318&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-135848520-null-null.142^v101^pc_search_result_base7&utm_term=gitlab%E5%8F%8C%E9%87%8D%E8%AE%A4%E8%AF%81&spm=1018.2226.3001.4187)

## 2.11 跑通代码

### 跑通代码

1. 配置好go 1.18环境  

```shell
go env -w GO111MODULE=on && \
go env -w GOPROXY=https://goproxy.cn,direct && \
git config --global url."git@team.applychart.com:sosobtc-backend/".insteadOf "https://team.applychart.com/lab/sosobtc-backend/" && \
go env -w GOPRIVATE=team.applychart.com
```

1. 注意首先需要team.applychart.com/lab/sosobtc-backend/go-core-lib 仓库的权限，以便于使用go-core-lib库
2. 安装依赖

```shell
go get -v
```

1. 本地启动项目需在main.go中注释掉kafka和ws客户端

```go:main.go
// "aicoin/pc-api/internal/ws_clients"
// "aicoin/pc-api/internal/consumer"
// 注释掉kafka和ws客户端
// 启动交易ws客户端
// ws_clients.RunTradeWsClient()
// 启动kafka消费者
// consumer.RunSlideWindowConsumerGroup()
```

1. 本地测试时，需要将main.go中的请求和响应的加密解密中间件注释掉

```go:main.go
// app.UseGlobal(middleware.DecryptRequestBody) 
// app.DoneGlobal(middleware.EncyrptReponseBody) 
```

同时需要把相应路由实现下的CheckLogin中间件注释掉,如route/account.go文件里面

```go:route/account.go
//baseAccount.Use(middleware.CheckLogin)     
```

1. 启动项目

```shell
go run main.go
```

1. 本地访问接口

```shell
http://localhost:8000
```

例如获取要闻作者精选文章的请求

```shell
curl -X POST 'http://localhost:8000/api/upgrade/article/memberArticles' \
-H 'Content-Type: application/json' \
-d '{
    "id": 123
}'
```

响应结果:

```json
{"success":true,"message":"","status":200,"data":{"count":0,"list":[]}}
```

## 逆天 取消代理后 composer install 成功了

## 2.12 PHP跑通 端口迁移

- `quote`指的是`行情`
- `global`全局
- 迁移的端口，路由路径应该保持不变
- 不用管繁体字
- composer update 出现了网络问题 关了代理就好了 逆天
- contorller的作用：参数校验、业务逻辑、返回数据
- MAC快捷键
    - 打开终端：`command + space`，输入`terminal`，回车
    - 删除整行：`command + delete`
    - 删除光标到行首：`control + u`  
  
- MAC-Redis

    ```shell
    brew install redis
    brew services start redis
    redis-cli ping
    ```

- `cp source target` 复制文件
- `cp -r source target` 复制文件夹
- `mv source target` 移动文件
- `mv source1 source2` 移动文件并重命名
- `rm source` 删除文件
- `rm -r source` 删除文件夹
- `rm -rf source` 强制删除文件夹
- Mac截图 `command + shift + 4 + control` 截图
- PHP 
    - 单例模式 (Singleton Pattern) 是一种软件设计模式，它的目的是确保一个类只有一个实例，并提供一个全局访问点来访问它。
    - 工厂模式 (Factory Pattern) 是一种创建型设计模式，它提供了一种创建对象的接口，但由子类决定要实例化的类是哪一个。

### Redis

Redis 是一个开源的内存数据结构存储系统，用作数据库、缓存和消息代理。它支持多种数据结构，如字符串、哈希、列表、集合、有序集合、位图、HyperLogLogs 和地理空间索引等。以下是一些 Redis 的快速入门知识和操作：

基本概念
- Key-Value Store：Redis 是一种键值型存储数据库，通过键来访问存储在内存中的数据。
- 持久性：虽然 Redis 是以内存为主的数据存储，但是它可以通过数据快照（snapshots）和日志文件持久化数据到硬盘。
- 发布/订阅：支持消息订阅和发布功能，适用于消息队列的场景。
- 客户端连接：你可以通过多种语言的客户端与 Redis 进行交互。

什么时候使用 Redis
- 高性能需求：因 Redis 数据存储在内存中，数据读写非常快速，适用于高实时性要求的场景。
- 数据缓存：经常用作 Web 应用的缓存层，减少数据库负载。
- 计数器：天然适合用于页面浏览计数、点赞等功能场景。
- 消息队列：可用于轻量级消息队列系统。

安装 Redis

```shell
brew install redis
```

### WebSocket vs. HTTP

- WebSocket 是双向的，HTTP 是单向的。
- WebSocket 是全双工的，HTTP 是半双工的。
- WebSocket 是长连接的，HTTP 是短连接的。
- 必须通过HTTP请求启动WebSocket握手过程。如果握手成功，HTTP连接将被转换为WebSocket连接；如果失败，则保持为普通的HTTP交互或者尝试其他形式的通信方式。

WebSocket 适用场景：

- 实时性要求高的应用：如在线交易系统、体育赛事直播、在线教育互动课堂等。
- 需要频繁交换小量数据的场景：例如聊天应用、社交媒体的通知提醒等。
- 多人协作工具：如Google Docs这样的在线文档编辑服务，允许多个用户同时查看和编辑同一个文档。

### 消息队列 vs. go channel

#### Go Channels

- **进程内通信**：Go channels主要用于同一个进程内的goroutine之间的通信。它们提供了一种简单且类型安全的方式来同步数据和协调goroutine的工作。
  
- **轻量级**：由于channels是在同一个进程内存空间内操作，因此开销较小，非常适合于解决并发编程中的同步问题。

- **使用方便**：Go语言内置了对channels的支持，使得开发者可以很容易地在代码中创建、发送和接收消息。

#### 消息队列（如Kafka, RabbitMQ等）

- **跨进程/跨机器通信**：消息队列允许不同进程甚至不同机器上的应用进行通信。这使得它成为构建分布式系统的关键组件之一。

- **持久化支持**：大多数消息队列服务提供了消息持久化的功能，这意味着即使消费者暂时不可用，消息也不会丢失。这对于需要高可靠性的场景非常重要。

- **扩展性和容错性**：消息队列通常设计为可水平扩展的架构，并且具备良好的容错能力。例如，Kafka可以在多个节点上复制消息，确保即使某些节点失败也不会影响系统的正常运行。

- **复杂的消息路由和过滤**：消息队列系统提供了丰富的机制来控制消息如何被分发给不同的消费者，包括主题订阅、广播、基于内容的路由等。

- **多种协议支持**：与仅限于Go语言环境的channels不同，消息队列可以通过多种协议（如AMQP, MQTT, STOMP）与各种语言编写的应用程序交互。

- **监控和管理工具**：现代的消息队列系统通常配备有强大的管理和监控工具，可以帮助开发者更好地理解和优化他们的消息处理流程。

#### 总结

虽然Go channels和消息队列都能用于实现异步通信，但它们适用的场景有所不同。Channels更适合于同一程序内部或同一服务器上的goroutine间通信，而消息队列则适用于更广泛的分布式环境下的应用程序间通信。如果你正在开发一个需要跨网络边界传递消息的分布式系统，那么使用像Kafka这样的消息队列会比单纯依赖Go channels更加合适。反之，在单一进程中协调goroutine时，Go channels则是更为直接和高效的解决方案。

消息队列中的传输对象并不局限于JSON格式，它可以是任何形式的数据序列化格式。

## 2.13 本地测试 测试环境测试

- 一到公司就问旷哥 `代码的位置、命名` 合不合适。 
- curl 
```shell
curl -X POST 'http://localhost:8000/api/upgrade/article/memberArticles' \
-H 'Content-Type: application/json' \
-d '{
    "id": 123
}'
```         
- DevTools
- git插件 thunder client插件
- k8s ingress 看ci.yml来确定哪些ingress.yaml需要修改。
- 公司文档要用公司邮箱注册登陆。 
- Conda是包管理系统和资源管理系统。Anaconda包含了Python及其包管理工具Conda及流行库。
- CUDA 则与图形处理和高性能计算密切相关。
- curl 是一个用于在命令行中与网络进行交互的工具。它的全名是 "Client URL"。这个工具能够通过多种网络协议（如 HTTP、HTTPS、FTP 等）进行数据传输。
- `-v`：显示详细的请求和响应信息。  
- `-X, --request <command>`：指定请求方式，如 GET、POST、PUT 等。
- `-d, --data <data>`：发送 POST 数据。
- `-H, --header <header>`：添加 HTTP 请求头。
- `-u, --user <user:password>`：用于基本的 HTTP 身份验证。
curl下载文件：
```shell
curl -O https://example.com/file.txt
curl -o myfile.txt https://example.com/file.txt 
```
- 赋予文件可执行权限
```shell
# change mode to executable
chmod +x filename 
```
- 执行文件
```shell
./filename
```
- 查看文件内容
```shell
cat filename
```

- Mac查看处理器架构
```shell
uname -m
```
### git
有空学习一下git图形化
命令行操作：
```shell
git clone https://github.com/aicoin/pc-api.git

# 在 master 分支，拉取最新代码
git checkout master
git pull origin master

# 创建一个新分支
git checkout -b feature/your-feature-name  # 替换为功能名称 b是指branch

# 进行开发 提交代码
git add .
git commit -m "Add your feature description"  # 替换为具体的提交描述

# 推送到远程分支
git push origin feature/your-feature-name

# 在 GitLab 上创建一个 Merge Request 合并develop分支


```

### go代码格式化
```shell
gofmt -s -w ./... # 在 Go 语言中，... 是用来递归指定路径的，它会匹配当前目录下以及所有子目录中的 .go 文件。
```

### CI/CD
CI/CD 是“持续集成”（Continuous Integration）和“持续交付”（Continuous Delivery）或“持续部署”（Continuous Deployment）的缩写，代表了一种软件开发实践，旨在提高软件开发的效率和质量。我们来逐一解释这两个概念：

#### 1. 持续集成（CI）

定义：持续集成是一种软件开发实践，开发人员将代码频繁地（通常是每天多次）合并到主分支。每当代码合并后，自动化测试和构建过程会立即运行，以确保新代码没有引入错误。

好处：

提早发现并修复错误。
保持代码库的健康和稳定。
使团队成员之间能够快速共享工作进展。

#### 2. 持续交付（CD）

定义：持续交付是指在持续集成的基础上，确保随时可以将最新的代码部署到生产环境。每次合并后的构建版本都是可部署的，通常会通过自动化脚本和工具进行测试和预部署。

好处：

降低了发布时的风险。
用户能够更快地使用到新功能。
开发人员能更快地从用户反馈中得到学习。

#### 3. 持续部署（Continuous Deployment）

定义：持续部署是持续交付的进一步延伸，意味着代码每次通过测试后都会自动部署到生产环境，而无需人工干预。这样一来，用户能实时接触到最新的功能和修复。

好处：

- 最大限度地减少了发布的时间延迟。
- 提高了反馈的速度。
- 合并后会自动启动 CI/CD 吗？
- 基于配置：是否会在合并后自动触发 CI/CD 流程，取决于项目的 CI/CD 配置和使用的工具。例如，常见的 CI/CD 工具如 Jenkins、GitLab CI/CD、Travis CI、CircleCI 和 GitHub Actions 等，都是可以配置为在代码合并（例如从一个开发分支合并到主分支）后自动执行 CI/CD 流程的。

一般流程：

- 开发者将代码推送到版本控制系统（如 Git）。
- 触发 CI 过程，运行自动化测试和构建流程。
- 如果通过所有测试，触发 CD 过程，自动将代码部署到测试环境或生产环境。

#### 总结

CI/CD 是现代软件开发流程中非常重要的一部分，它能够提高代码质量，加快发布时间。在合并代码后，CI/CD 流程是否会自动启动，通常是通过配置文件（如 .gitlab-ci.yml、Jenkinsfile、circleci/config.yml 等）进行设置的，确保代码的健康和高效交付。

### 路由转发

根据.gitlab-ci.yml文件，确定哪些yaml文件需要修改：

```yaml
ingress-dev:
  stage: ingress
  image: tke-registry.tencentcloudcr.com/public/docker-library:alpine-kubectl-tcie-aicoin-dev
  tags:
    - docker-runner
  only:
    - develop
    - /^dev-v[0-9]{8}$/
  script:
    - sed -i 's/_APP_NAME_/'"$APP_NAME"'/g' ./kubernetes/ingress_dev.yaml
    - kubectl --kubeconfig=$HOME/.kube/config apply -f ./kubernetes/ingress_dev.yaml
    - kubectl --kubeconfig=$HOME/.kube/config apply -f ./kubernetes/ingress_dev_login.yaml
```

修改对应的yaml文件：

```yaml
# 直接加到后面，复制上文，修改路由路径即可
# 注意ingress_dev.yaml两个host，需要分别修改
  - http:
      paths:
      - backend:
          service:
            name: api-pc
            port:
              number: 80
        path: /clientCheck
        pathType: Prefix
```

## 2.14 yapi

YApi不仅仅是管理与测试接口的工具，同时也提供了强大的文档编写、团队协作和Mock等功能，帮助团队更高效地进行API开发与测试。具体来说，YApi提供了以下几个核心功能：

- 接口管理：YApi允许开发团队创建和维护API文档，包括接口的描述、参数、返回值等信息。这使得团队成员能够清晰地了解接口的功能与使用方法。
- 接口测试：YApi提供了测试接口的功能，允许用户在界面上直接调用API并查看返回结果。这对于开发和测试环节都很有帮助，方便进行快速验证。
- 团队协作：YApi支持团队成员之间的协作，可以让开发人员、测试人员以及产品经理等各方共同参与接口的文档编写与维护，确保接口的准确性和一致性。
- 接口版本管理：YApi支持接口的版本管理，使得团队可以有效跟踪不同版本的接口变化。
- Mock功能：YApi还提供了Mock服务，允许开发者在后端未完成时模拟接口的返回数据，便于前端开发和测试。

## 2.17 MR

- 没有走加密中间件是因为少了`ctx.Next()`
- 不会解决冲突

## 2.18

- 在 Redis 中，所有的值都以字节数组的形式存储
- reflect使用
