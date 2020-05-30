# 目录结构

```bash
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── app  
│   ├── dependencies.php
│   ├── middleware.php
│   ├── repositories.php
│   ├── routes.php
│   └── settings.php
├── composer.json
├── composer.lock
├── docker-compose.yml
├── logs
│   └── README.md
├── phpstan.neon.dist
├── phpunit.xml
├── public
│   ├── index-test.php
│   ├── index.php
│   └── web
├── runtime
│   ├── access.log
│   └── error.log
├── src
│   ├── Application
│   ├── Domain
│   └── Infrastructure
├── tests
│   ├── Application
│   ├── Domain
│   ├── Infrastructure
│   ├── TestCase.php
│   └── bootstrap.php
├── var
│   └── cache
└── vendor
```

app  配置目录，设置配置，中间件配置，路由配置，依赖配置

public  公共入口  index.php 和 前端文件，部署分离可单独拆分

src 后端代码 

```bash
├── Application
│   ├── Actions
│   ├── Controller
│   ├── Handlers
│   ├── Middleware
│   └── ResponseEmitter
├── Domain
│   ├── DomainException
│   └── User
└── Infrastructure
    └── Persistence
```

Application 应用控制层，中间件层，Handlers 层

Domain  领域模型层

Infrastructure  基础设施  持久化层，邮件发送，消息通知

Service 服务层 



