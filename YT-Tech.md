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
git checkout -b feature-name  # 替换为功能名称 b是指branch

# 进行开发 提交代码
git add .
git commit -m "Add your feature description"  # 替换为具体的提交描述

# 推送到远程分支
git push origin feature-name

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
- 发版 = 上线
- 合并到master != 部署
- 本地只用安装navicat等工具即可,不需要安装数据库
- 接口迁移小心数据结构
- 在 Kubernetes 集群中，Ingress 是一种 API 对象，它管理外部访问集群内部服务的路由。Ingress 可以将外部请求根据不同的路由规则路由到所需的服务。以下是有关 Kubernetes Ingress 的一些关键点，帮助您理解它在接口迁移中的作用：

Ingress 的作用


外部访问控制:


Ingress 允许您管理对 Kubernetes 服务的外部访问策略。这意味着您可以指定如何将外部请求路由到集群中的具体服务（通常是提升服务的可用性和安全性）。



路由转发:


Ingress 文件中的路由转发规则设置了请求如何被转发到不同的后端服务（在您的 caso中，这可能涵盖老的 PHP 服务和新的 Go 服务）。您可以根据请求的路径或主机名定义如何路由请求。



负载均衡:


Ingress 还能提供负载均衡功能，分发流量到多个后端服务实例，以提高可用性和性能。



TLS/SSL 终止:


Ingress 允许您配置 TLS，提供 HTTPS 支持，确保与客户端之间的安全通信。



基于主机名和路径的路由:


您可以根据请求的主机名和 URL 路径来决定流量应该转发到哪个具体服务。例如，假设有这样一个场景：

所有对 /api/v1/ 的请求被转发到新的 Go 服务。
所有对 /legacy/ 的请求被转发到老的 PHP 服务。





示例 Ingress 资源
以下是一个简单的 Ingress 资源定义示例，演示了如何将流量路由到不同的服务：

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api/v1/
        pathType: Prefix
        backend:
          service:
            name: go-service
            port:
              number: 80
      - path: /legacy/
        pathType: Prefix
        backend:
          service:
            name: php-service
            port:
              number: 80

在这个例子中：


所有请求 example.com/api/v1/ 会被转发到名为 go-service 的服务。
请求 example.com/legacy/ 则会被转发到名为 php-service 的服务。

在接口迁移中的重要性
在您的接口迁移过程中：


增加灵活性: 通过 Ingress，您可以同时保留老的 PHP 接口与新的 Go 接口，逐步迁移，而无需立即关闭旧的服务。
平滑过渡: 您可以逐步将流量从 PHP 服务转移到 Go 服务，进行 A/B 测试，方便监测新代码的表现和稳定性。
管理复杂性: 随着系统的演进，Ingress 能帮助您简化不同服务之间的路由管理，降低维护复杂度。

总结
Kubernetes Ingress 文件中的路由转发功能在您的项目中起到了至关重要的作用。它不仅允许您控制和管理外部请求的路由，还有助于实现平滑的服务迁移与过渡。希望这能帮助您更好地理解 Ingress 在接口迁移过程中的作用！如果您有更多问题，欢迎继续提问。

- Ingress 是 Kubernetes 中的一种 API 资源，主要用于管理外部访问到 Kubernetes 集群内部服务的访问控制和路由。它提供了一种灵活的方式，可以在 K8s 集群中定义和描述如何将外部 HTTP(S) 请求路由到内部服务。下面是关于 Ingress 的一些关键点：

1. 基本概念

定义外部访问: Ingress 用于定义如何访问在 Kubernetes 集群内运行的服务，它允许将外部 HTTP(S) 请求映射到相应的服务。
路由规则: 以 URL 路径或主机名的方式定义路由规则，通过这些规则，Ingress 能够决定将请求转发到哪个具体的后端服务。

2. Ingress 的组件

Ingress Controller: Ingress 本身是一个定义，但实际上将请求路由到后端服务的是 Ingress Controller。其实，有多种 Ingress Controller 实现，如 NGINX、Traefik、HAProxy 等。Ingress Controller 监视 Ingress 资源的变化，并自动执行路由。

3. 功能特性

负载均衡: Ingress 可以将流量均匀地分配到多个 Pods。
TLS/SSL 支持: 可以配置 TLS 证书以支持 HTTPS。
基于路径和主机的路由: 通过定义不同的路径和主机名，可以将请求路由到不同的后端服务。
定义重定向和重写: 可以根据需要指定请求重定向或 URL 重写规则。
中间件支持: 一些 Ingress Controllers 支持自定义中间件，用于处理认证、流量控制等。

4. 示例
以下是一个简单的 Ingress 定义示例：

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api/v1/
        pathType: Prefix
        backend:
          service:
            name: my-api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-frontend-service
            port:
              number: 80

在这个例子中，所有对 example.com/api/v1/ 的请求将被路由到名为 my-api-service 的服务，而对 example.com/ 的请求则将被路由到 my-frontend-service 服务。

5. 使用场景

将不同的微服务暴露给外部用户。
提供 API 网关功能，集中管理多个服务的外部流量。
在服务迁移或升级时，为旧服务和新服务设置路由。

总结
Ingress 是 Kubernetes 中关键的流量管理功能，能够有效地将外部请求路由到正确的服务，并提供负载均衡和安全性。通过合理配置 Ingress，可以方便地管理和使用 K8s 集群内部的微服务架构。

- DESC table_name;
- tke是什么？

## 2.19 

- 使用快捷键‌：在Windows上按Ctrl + ,，在Mac上按Cmd + ,，可以直接打开VSCode设置页面。
- `"workbench.editor.enablePreview": false` - 这将确保在打开新文件时，之前的文件不会被关闭。
- git 操作小技巧 ： stash 暂存更改，以后可以直接应用
- checklogin中间件需要token
- yapi 请求参数不能手动输入，需要复制
- git push -u origin fix-search-cache 设置上游 如果远程没有fix-search-cache这个分支的话，就会自动创建么
- 大表建索引 会崩
- EXPLAIN分析慢查询
- develop一定要合！！！

- 从堆栈追踪（stacktrace）部分，可以看到多个函数之间的调用关系。在您提供的调用栈中，按照从上到下的顺序，首先是处理错误的闭包，然后是导致错误的函数，然后是调用这个函数的函数，直到下方的主函数 main.main。
- 在 Go 语言的堆栈追踪中，每个堆栈帧一般包含以下信息:包名和函数名、文件名和行号。
- recover 只能捕获 panic，而不能捕获一个普通的 error。在 Go 中，panic 是一种用于处理程序错误的机制，用来立即停止当前函数的执行，并开始逐层向上返回，直到程序退出或者被 recover 捕获。
- 在 Go 中，通常建议将 recover 的匿名函数放在函数的开始处，这样可以确保在发生 panic 时能立即捕获到它。
- recover 必须与 defer 结合使用才能有效捕获 panic。recover 是一个内置函数，其主要作用是从 panic 中恢复控制并继续执行程序。当程序发生 panic 时，当前的执行流会被清空，而控制权会被转移到最近的带有 defer 的函数中。只有在该 defer 的执行上下文中调用 recover，才能捕获到 panic 的信息。
- 在 Git 中，git merge 命令用于将一个分支的更改合并到当前分支。如果当前分支是 b1，并且你执行了 git merge b2，以下步骤和结果将会发生：

1. 合并过程

合并分支：Git 会尝试将 b2 分支的更改合并到当前分支 b1。
快进合并与三方合并：

快进合并（Fast-Forward Merge）：如果 b1 是 b2 的祖先（即 b2 是在 b1 分支的基础上创建的），则会执行快进合并。在这种情况下，Git 只需将 b1 指针移动到 b2 的最新提交。你会在历史中看不到合并提交。
常规合并：如果 b1 和 b2 之间有不同的提交（即两者都有各自的提交历史），则会通过创建一个新的合并提交（merge commit）来整合这两个分支的更改。这个新的合并提交将有两个父提交：b1 的最新提交和 b2 的最新提交。



2. 合并冲突

在合并的过程中，如果 b1 和 b2 对同一文件的同一部分进行了不同的更改，Git 将无法自动合并这些更改。这种情况下，Git 会标记为冲突，允许你在解决冲突之后再完成合并。
你可以使用 git status 查看冲突文件，并手动编辑冲突段落，完成后用 git add  标记文件为已解决，并使用 git commit 完成合并。

3. 合并结果

如果没有冲突：合并将成功，新的提交将被创建。如果是快进合并，将直接更新 b1 分支指向 b2 的最新提交。
如果有冲突：你需要解决冲突后，才能完成合并。

示例步骤
假设当前在 b1 分支上，执行 git merge b2：



无冲突且快进合并：

git checkout b1
git merge b2
# `b1` 移动到 `b2` 的最新提交



无冲突且创建合并提交：

git checkout b1
git merge b2
# 创建合并提交，合并了 `b1` 和 `b2` 的最新更改



有冲突：

git checkout b1
git merge b2
# Git 会提示冲突，必须手动解决



总结
执行 git merge b2 时，b2 的更改将被合并到当前的 b1 分支。具体的合并方式（快进还是创建合并提交）以及是否产生冲突，取决于这两个分支的提交历史和更改内容。在执行 git merge b2 命令时，b2 分支本身不会被删除或修改。

- 慢查询（Slow Query）是指在数据库操作中，执行时间超过设定阈值的查询。
- 在 GORM（一个流行的 Go 语言 ORM 框架）中，Trace 方法会在每次执行 SQL 查询时被自动调用。这里一点很重要，就是 GORM 提供了一种机制，用于在执行数据库操作时进行自定义行为，比如日志记录、性能监控等，这通常是通过实现特定的接口来实现的。
- context.Context 是 Go 语言中标准库提供的一个类型，用于在程序中传递上下文信息。它的主要作用是携带跨 API 边界的请求范围（request-scoped）、处理控制和取消信号以及请求的时间限制等信息。以下是对 context.Context 的详细解释：

主要用途


请求范围：


context.Context 允许你在不同的 goroutine 中传递请求范围的值。比如，这个值可以是请求的 ID、用户身份、处理结果等。
例如，HTTP 处理程序可以将用户的身份信息放入上下文中，然后在处理请求的过程中，可能会启动其他的 goroutine，在这些 goroutine 中也能访问到这个上下文信息。



取消信号：


上下文可以用来控制 goroutine 的取消。通过在上下文中设定 Done() 方法的取消信号，相关的 goroutine 可以在必要的时候停止执行。
例如，假设一个请求是基于用户的操作发起的，如果用户在操作期间中断了请求，你可以使用上下文来取消与该请求相关的所有 goroutine。



超时与截止日期：


上下文还允许你定义超时和截止日期。当时间到达后，任何使用该上下文的操作都会收到取消信号。
例如，如果你希望某个数据库查询在 10 秒内返回结果，你可以创建一个带有超时的上下文，以确保查询不会无休止地等待。



例子
下面的例子展示了如何使用 context.Context 来控制请求的生命周期，以及在 goroutine 中使用它。

package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 创建一个带有超时的上下文
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel() // 确保释放资源

    go func() {
        // 模拟一个长时间运行的操作
        time.Sleep(3 * time.Second)
        fmt.Println("Completed long operation")
    }()

    select {
    case <-ctx.Done():
        fmt.Println("Operation cancelled:", ctx.Err())
    }
}

在这个例子中：


context.WithTimeout 创建了一个上下文，设置了 2 秒的超时限制。
启动了一个 goroutine，它会执行一个 3 秒的操作。
当运行时间超过 2 秒时，主 goroutine 会收到上下文的取消信号，并通过 ctx.Done() 得知操作已被取消，同时可以访问 ctx.Err() 来获取错误信息。

使用场景

HTTP 处理程序：在处理来自客户端的请求时，使用上下文可以传递请求范围的数据，如请求 ID，身份验证信息等。
数据库操作：在数据库查询中，可以使用上下文进行超时控制，确保操作不会无休止地等待。
并发处理：在并发程序中，使用上下文传递取消信号和请求信息，有助于管理多个 goroutine 之间的交互。

总结
context.Context 是一个很有用的工具，它能够在 Go 的并发编程中提供请求范围的控制和数据管理。它帮助开发者更清晰地组织代码，并在运行时对操作进行控制，适应复杂应用程序中的各种需求。使用上下文有助于保持程序的清晰度和可维护性。

- SHOW INDEX FROM script_strategy_detail;
- 堡垒机（Bastion Host）是一种安全机制，通常用于增强对关键系统（如生产环境数据库服务器）的访问控制与审计。它的主要作用是作为一个中间层或代理，允许用户通过堡垒机连接到后端的目标服务器，但对后端服务器的直接访问则被限制。

堡垒机的主要特点


安全控制：


堡垒机作为入口，所有的访问请求首先会通过它进行访问控制和身份验证，从而确保只有经过授权的用户才能访问下游资源。



审计与监控：


堡垒机可以记录用户的所有操作（如SSH登录、命令执行等），为后续的审计和监控提供详细的日志，以便事后分析和取证。



减少攻击面：


通过堡垒机接入后端服务器，避免了直接在生产环境中暴露数据库或关键服务，减少了潜在的安全风险。



集中管理：


建立一个集中管理的接入点，便于管理员管理和监控用户的访问权限和行为。



为什么连接生产环境的数据库需要堡垒机


增强安全性：


在生产环境中，数据库通常包含敏感信息，利用堡垒机可以大幅提升对数据库访问的安全性，防止未授权访问。



合规性：


在一些行业如金融、医疗等，对数据访问有严格的合规要求，堡垒机的使用可以帮助满足这些要求，提供必要的审计日志。



隔离网络：


堡垒机确保内网和外网的隔离，开发人员无法直接连接到生产数据库，而是通过堡垒机这样的中介进行访问，进一步保护网络架构。



控制访问权限：


通过堡垒机可以更加灵活地管理用户权限，提供针对不同角色的不同访问策略，确保用户只能访问与其工作相关的资源。



减少人为错误：


通过堡垒机集中管理访问，可以减少不必要的人为错误和配置问题，避免在生产环境中直接执行不当操作。



简化访问管理：


管理员可以在堡垒机上配置、管理和删除用户访问，而不需要单独在每台机器上进行相应配置，简化了管理过程。



总结
堡垒机是连接生产环境数据库的重要安全措施，能够大幅提升系统的安全性、合规性和管理效率，是现代IT架构中不可或缺的一部分。通过使用堡垒机，可以有效地保护关键数据，防止数据泄露和未授权访问，提升组织的整体安全防护水平。

- EXPLAIN 是 MySQL 中用于分析 SQL 查询性能的重要工具。通过 EXPLAIN 命令，用户可以查看查询的执行计划，从而了解 MySQL 将如何处理该请求。这对于性能调优和优化查询非常有帮助。
- id_rsa 和 id_rsa.pub 是用于 SSH（Secure Shell）协议的密钥文件的常见命名约定，通常用来进行公钥认证。其中的 id 指代身份（identity），而 rsa 则指代所使用的加密算法（RSA算法）。
- 月活：
SELECT COUNT(*) AS user_count
FROM user
WHERE lasttime >= NOW() - INTERVAL 1 MONTH;

- 是的，在 SQL 中，DESC 可以用于两种不同的上下文中，具体取决于它的使用方式
- 同一个数据库，不同的表可以用不同的引擎
- mysql查询语句，查询时间和获取时间区别
- 在 MySQL 中，如果你希望强制某个查询使用特定的索引，可以使用 FORCE INDEX 语法。这可以帮助你在某些情况下优化查询性能，特别是当 MySQL 查询优化器选择了不理想的索引时。
- 你可以将 MySQL 数据库中的数据同步到 Elasticsearch（ES）中，以提高搜索性能和数据访问速度。使用 Elasticsearch 进行查询可以大幅度提高在大数据集上的搜索效率，特别是在需要进行复杂查询或全文搜索时。
- 在 全文搜索、复杂查询和 大规模数据处理 场景下，Elasticsearch 通常会比 MySQL 快。
在涉及 高事务处理、复杂 join 查询、或需要严格 数据一致性 的场景下，MySQL 可能更合适。
Elasticsearch 的数据主要存储在磁盘上，但也会利用内存来提高查询性能和响应速度。

- 堡垒机 给运维公钥 然后常规和SSH都要填写
- 和PC和测试对接

## 2.20

### MySQL

```bash
brew install mysql
brew services start mysql
```

### command + r 刷新
### command + shift + p VSCode 命令面板

## 2.21

### 1.18 版本go 安装插件太麻烦了，升级

使用 goenv 工具来升级 Go 版本非常简单。以下是详细的步骤：

1. 查看可用的 Go 版本
首先，您可以查看可用的 Go 版本列表，确认您想要升级到的版本：

goenv install -l

这将列出所有可用的 Go 版本，您可以选择一个目标版本。

2. 安装新版本
选择一个您希望安装的 Go 版本，比如假设您要安装 1.20.5，可以使用以下命令：

goenv install 1.20.5

这个过程可能会持续几分钟，具体时间取决于网络速度和您的计算机性能。

3. 设置全局或局部版本
安装完成后，您可以设置新安装的 Go 版本为全局使用的版本：

goenv global 1.20.5

或者，如果您只想在特定的项目中使用该版本，可以在项目目录中使用：

cd /path/to/your/project
goenv local 1.20.5

这样会在您的项目文件夹中创建一个 .go-version 文件，指定该项目使用的 Go 版本。

4. 验证升级
最后，您可以通过以下命令确认 Go 版本已成功升级：

go version

这应该会显示您刚安装的新版本。

小结
通过上述步骤，您可以使用 goenv 工具方便地升级 Go 版本。如果您在执行过程中遇到任何问题，欢迎随时提问！

### 插件
go install golang.org/x/tools/gopls@latest         # Language Server from Google
go install github.com/cweill/gotests/...@v1.6.0   # Generate unit tests

go install github.com/fd/sort+tags/...@v1.17.0      # Modify tags on structs
go install github.com/impl@v1.4.0                    # Stubs for interfaces

go install golang.org/x/play@v1.0.0                  # The Go playground
go install github.com/go-delve/delve/cmd/dlv@latest  # Go debugger (Delve)
go install honnef.co/go/tools/cmd/staticcheck@latest # Linter


### 
(base) applychartdeiMac-2:YJM applychart$ chmod a+w ~/.bash_profile
chmod: Unable to change file mode on /Users/applychart/.bash_profile: Operation not permitted
(base) applychartdeiMac-2:YJM applychart$ sudo chmod a+w ~/.bash_profile
Password:

### tmd zsh和bash都要配置
哪VSCode用的是什么？

export PATH="$HOME/.goenv/shims:$HOME/.goenv/bin:$PATH"

# 对于 Bash
source ~/.bash_profile   # 或 source ~/.bashrc

# 对于 Zsh
source ~/.zshrc



### 
从您提供的 PATH 输出来看，/Users/applychart/.goenv/shims 实际上在路径的后面部分，而 /Users/applychart/.goenv/versions/1.18.10/bin 仍然出现在 PATH 的前面。这就是为什么在执行 go version 时仍然指向 1.18.10 的原因。让我们解决这个问题。

解决步骤
1. 更新 PATH 设置
您需要确保 /Users/applychart/.goenv/shims 在 PATH 的最前面，且 1.18.10/bin 不在 PATH 中。为了做到这一点，请打开您的 shell 配置文件（例如 ~/.bash_profile, ~/.bashrc 或 ~/.zshrc），并确保将以下内容放置在文件的最前面：

export PATH="$HOME/.goenv/shims:$HOME/.goenv/bin:$PATH"

确保没有多余的 versions/1.18.10/bin 路径。

2. 重新加载配置文件
保存更改后，重新加载您的 shell 配置文件：

# 对于 Bash
source ~/.bash_profile   # 或 source ~/.bashrc

# 对于 Zsh
source ~/.zshrc

3. 确认 PATH 更新
再次检查您的 PATH 变量，确保 /Users/applychart/.goenv/shims 在最前面，而不再包含 /Users/applychart/.goenv/versions/1.18.10/bin：

echo $PATH

4. 强制重新哈希
然后，使用以下命令强制重新哈希 goenv：

goenv rehash

5. 检查 Go 版本
最后，检查 go 命令的路径和版本，以确保现在指向 1.23.6：

which go
go version

如果一切顺利，which go 输出应该是 /Users/applychart/.goenv/versions/1.23.6/bin/go，并且 go version 输出应该是 go version go1.23.6 ...。

总结
经过以上步骤后，您应该能够成功切换到 Go 1.23.6，并解决版本指向不正确的问题。如果仍然存在问题，请继续告知我，我们可以进一步排查。

### 妈的 重启VSCode在能用命令面板安装插件

ls $(go env GOPATH)/bin

### git fetch origin 
git checkout -b refactor-client-check origin/refactor-client-check

### fetch vs. pull

git fetch 和 git pull 都是用于与远程 Git 仓库交互的命令，但它们之间存在一些重要的区别。下面是对这两个命令的详细解释：

1. git fetch

功能：fetch 命令从远程仓库获取所有的更新内容（例如，分支、标签、提交等），但不会自动合并到当前的工作分支。
用法：git fetch origin


效果：

获取更新后，你可以查看远程仓库的状态、比较更改或检查新的提交。
你的当前分支不会受到影响，仍然保持不变。此时，你需要手动合并更改（使用 git merge）或者切换到新的分支使用（可以用 git checkout）。


适用场景：当你想首先查看远程更新的内容，然后再决定是否要合并时，使用 git fetch 是一种比较安全的做法。

2. git pull

功能：pull 命令实际上是 fetch 和 merge 的组合。它从远程仓库获取更新并立即将这些更新合并到你的当前工作分支中。
用法：git pull origin main


效果：

从远程仓库获取最新的提交，并自动尝试将这些更改合并到你当前的分支。
如果合并成功，工作树将更新为包含新的更改。如果出现冲突，你需要解决冲突后才能完成合并。


适用场景：当你希望快速获取远程的更改并将其应用到当前分支时，git pull 非常方便。

总结

git fetch：只获取数据，且不会导致任何更改发生。适合在选择是否合并之前先查看远程的更新。
git pull：既获取更新又自动合并，适合快速更新当前分支。

在使用 git pull 时要注意，因为合并过程可能引入冲突，因此在复杂的工作流中，先使用 git fetch 来预览更新，再决定是否合并通常是一种更安全的方式。

