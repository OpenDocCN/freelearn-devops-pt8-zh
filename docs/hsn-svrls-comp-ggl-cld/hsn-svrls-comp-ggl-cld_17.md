# 第十三章：评估

# 第一章：介绍应用引擎

1.  Push Queue 和 Pull Queue 是任务队列支持的服务分发类型。

1.  Memcache 支持的两级服务是共享 Memcache 和专用 Memcache。

1.  Cloud Datastore 是一个托管的无模式（NoSQL）文档数据库。

1.  GAE 支持的语言：Python、Go、Node 和 PHP。

1.  GAE 支持的流量分割算法包括 IP 流量、Cookie 和随机。

1.  GFE 在 GAE 中的作用是通过自动扩展基础设施路由 Google 应用引擎流量。

1.  GAE 支持的三种扩展方式分别是自动扩展、基本扩展和手动扩展。

1.  任务队列机制用于将长期运行的工作负载与 HTTP 请求/响应生命周期分离，以提高效率。

# 第二章：使用应用引擎开发

1.  错误。

1.  GAE 提供的选项包括 IP 地址、Cookie 或随机选项。

1.  基本和高级是 Stackdriver Logging 中的日志过滤选项。

1.  错误。

1.  正确。

1.  `gcloud app deploy` 命令。

1.  `appspot.com` 域名。

# 第三章：介绍轻量级函数

1.  正确。

1.  延迟或推迟的任务是异步任务。

1.  正确。

1.  正确。

1.  `trigger-http` 命令。

1.  Stackdriver Logging。

1.  `gcloud functions deploy NAME --runtime RUNTIME TRIGGER` 命令。

1.  exports 函数是入口点——这表示 Cloud Functions 接口是公开的。

# 第四章：开发云函数

1.  Cloud Functions 运行在 `8080` 端口。

1.  主题触发器由 Cloud Pub/Sub 使用。

1.  Go、Node 和 Python 是 Google Cloud Functions 支持的运行时语言。

1.  成功的 HTTP 响应代码是 2xx。

1.  客户端错误的 HTTP 响应代码是 4xx。

1.  服务器端错误的 HTTP 响应代码是 5xx。

1.  CORS 的目的是允许从另一个域访问网页上的资源。

1.  HTTP 头 – 在头部名称中添加 `access-control-allow-origin`。

# 第五章：探索作为服务的函数

1.  Cloud Pub/Sub 支持推送/拉取订阅。

1.  直通处理，多发布者，多订阅者。

1.  GET、DELETE、POST 和 PUT 动词与 HTTP 相关。

1.  使用 GET 使用户数据可以在 URL 中访问。

1.  元数据属性包含与 Cloud Storage 对象相关的信息。

1.  服务器上的问题。

1.  是的，Google Cloud 支持 OAuth v2。

1.  不——消息将排队等待重试。

# 第六章：云函数实验室

1.  调用 Cloud Function 需要 `cloudfunctions.invoker` 权限。

1.  Google KMS 提供加密密钥的存储。

1.  `allUsers` 权限实际上意味着公开访问适用于所有人。

1.  `allAuthenticated` 权限实际上意味着公开访问适用于经过身份验证的用户。

1.  `--max-instances` 参数。

1.  `gcloud functions add-iam-policy` 命令。

1.  `gcloud iam service-accounts create` 命令。

1.  Bastion 主机是一个隔离的主机，用于限制网络的进出访问。

# 第七章：介绍 Cloud Run

1.  单体应用与微服务应用的区别在于，微服务是松耦合架构、隔离的、标准接口、更易调试。而单体应用通常是紧耦合的，调试困难。

1.  GFE 执行将流量路由到项目资源的功能。

1.  两种同步事件处理模式：请求/响应、Sidewinder。

1.  `ENTRYPOINT` 关键字用于在容器执行时运行命令。

1.  `docker build -t NAME .` 命令。

1.  容器注册表是 Google Cloud 用于镜像管理的产品。

1.  自动化。

1.  Knative（中间件）位于应用程序与 K8s 之间。

1.  开放容器倡议（OCI）——标准化的容器接口。

1.  Debian、Ubuntu 和 Windows 是支持容器的一些操作系统。

# 第八章：使用 Cloud Run 开发

1.  基础 URL 是访问服务的端点。

1.  API 版本控制非常重要，因为它支持多个版本的部署。

1.  GET、PUT、POST 和 DELETE 方法预计将在 REST API 中可用。

1.  当客户端发送的数据不正确时，返回 HTTP 状态码 4xx 是合适的。

1.  Cloud Build 的最大持续时间为 600 秒。

1.  正确。

1.  Cloud Build 错误将在 Cloud Build 仪表板（概览）和历史记录（详细视图）中显示。

# 第九章：使用 Cloud Run for Anthos 开发

1.  在使用 Cloud Run for Anthos 时，支持的机器定制类型包括操作系统、GPU、CPU。

1.  使用 Cloud Run for Anthos 时，SSL 证书是手动配置的。

1.  部署到 Cloud Run for Anthos 时，`--platform gke` 平台标志是必需的。

1.  使用 Cloud Run for Anthos 时，端口 `8080` 用于服务访问。

1.  正确。

1.  Cloud Run for Anthos 是使用 Cloud Run 所必需的。

1.  `kubectl` 命令。

1.  Pod 是 Kubernetes 中的副本单位。

1.  GKE 支持通过 Ingress 处理外部流量。

# 第十章：Cloud Run 实验室

1.  使用 Container Registry 而非其他注册表的一个关键优势是 Container Registry 位于 Google 网络内。

1.  在命令行脚本中使用如 `$GOOGLE_CLOUD_PROJECT` 这样的环境变量非常有用。

1.  Cloud Build 支持分支和标签类型。

1.  `@cloudbuild.gserviceaccount.com` 命令。

1.  Dockerfile 和 `Cloudbuild.yaml` 得到 Cloud Build 的支持。

1.  Let's Encrypt 提供 Certbot SSL 证书。

# 第十一章：构建 PDF 转换服务

1.  `gsutil` 命令。

1.  Stackdriver Logging 在与日志相关的问题出现时最为有用。

1.  Docker 清单文件的目的是为要构建的镜像提供规范。

1.  Cloud Build 在创建时将镜像存储在容器注册表中。

1.  正确。

1.  使用带有 `-m` 参数的 `gsutil` 命令进行 Cloud Storage 的多区域请求。

1.  调用服务所需的权限类型是 `invoker`。

# 第十二章：通过 REST API 消费第三方数据

1.  Google Cloud 用于异步消息传递的产品是 Cloud Pub/Sub。

1.  Google Cloud Storage 支持的通知包括 finalize、delete、archive 和 metadataUpdate。

1.  对于新服务/产品的测试，GCloud SDK 提供了 beta 命令。

1.  正则表达式（Regex）是用于指定文本的常规表达式/简写形式。

1.  正确。

1.  正确。

1.  Stackdriver Trace。
