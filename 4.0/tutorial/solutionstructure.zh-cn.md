# 解决方案结构

## 项目解决方案结构

```
Solution
├─common (Folder)
│  ├─Project.Abstractions (Library)
│  ├─Project.Client (Library)
│  ├─Project.Core (Library)
│  └─...
├─src (Folder)
│  ├─ServiceA (Folder)
│  │  ├─Project.ServiceA (Web)
│  │  ├─Project.ServiceA.Core (Library)
│  │  └─Project.ServiceA.Domain (Library)
│  ├─ServiceB (Folder)
│  │  ├─Project.ServiceB (Web)
│  │  ├─Project.ServiceB.Core (Library)
│  │  └─Project.ServiceB.Domain (Library)
│  └─...
├─test (Folder)
│     ├─Project.Testing (Testing)
│     └─...
└─tools (Folder)
      ├─Project.DataTools (Console)
      └─...
```

## 项目说明

### Project.Abstractions
此项目一般提供通用数据模型。例如Api返回类型、Api状态枚举、通用查询对象等。
> 包引用
> * `Wodsoft.ComBoost.Data.Core` 使用ComBoost提供的模型

### Project.Core
此项目一般提供内部通用数据模型、通用服务接口。例如实体基类。
> 项目引用
> * `Project.Abstractions`

### Project.Client
可选项目。提供封装好的Api调用客户端给第三方项目使用。
> 项目引用
> * `Project.Abstractions`

### Project.ServiceA.Core
此项目一般提供`ServiceA`对外暴露的领域服务接口与领域服务事件、数据模型，供其它解决方案内的项目引用。
> 项目引用
> * `Project.Core`
> 
> 包引用
> * `Wodsoft.ComBoost.Core`

### Project.ServiceA.Domain
此项目一般实现`ServiceA`的领域服务、实体模型、数据模型。
> 项目引用
> * `Project.ServiceA.Core`
> 
> 包引用
> * `Wodsoft.ComBoost.Data` 
> * `Wodsoft.ComBoost.EntityFrameworkCore` 使用EF Core用于实体仓储
> * `Pomelo.EntityFrameworkCore.MySql` 使用MySql作为EF Core数据库
> * `Microsoft.EntityFrameworkCore.SqlServer` 使用SqlServer作为EF Core数据库
> * `AutoMapper.Extensions.ExpressionMapping` 使用实体仓储并通过AutoMapper映射至数据对象
> * `AutoMapper.Extensions.Microsoft.DependencyInjection` 使用实体仓储并通过AutoMapper映射至数据对象

### Project.ServiceA
此项目作为`ServiceA`的Web服务，一般提供HTTP与GRPC服务。  
在Startup中配置`ComBoost`与其它服务依赖。

> 项目引用
> * `Project.ServiceA.Domain`
> 
> 包引用
> * `Wodsoft.ComBoost.Mvc` 使用Asp.Net Core Mvc上下文作为领域上下文
> * `Wodsoft.ComBoost.Grpc.AspNetCore` 对其它项目提供基于Grpc的领域服务
> * `Wodsoft.ComBoost.Grpc.Client` 使用其它项目提供的基于Grpc的领域服务
> * `Wodsoft.ComBoost.Distributed.RabbitMQ` 使用RabbitMQ作为分布式领域事件提供器
> * `Microsoft.EntityFrameworkCore` 使用EF Core用于实体仓储
> * `Microsoft.EntityFrameworkCore.Tools` 使用EF Core用于实体仓储

### Project.ServiceB.Core
此项目一般提供`ServiceB`对外暴露的领域服务接口与领域服务事件、数据模型，供其它解决方案内的项目引用。
> 项目引用
> * `Project.Core`
> 
> 包引用
> * `Wodsoft.ComBoost.Core`

### Project.ServiceB.Domain
此项目一般实现`ServiceA`的领域服务、实体模型、数据模型。
> 项目引用
> * `Project.ServiceA.Core` 使用ServiceA的服务
> * `Project.ServiceB.Core`
> 
> 包引用
> * `Wodsoft.ComBoost.Data` 
> * `Wodsoft.ComBoost.EntityFrameworkCore` 使用EF Core用于实体仓储
> * `Pomelo.EntityFrameworkCore.MySql` 使用MySql作为EF Core数据库
> * `Microsoft.EntityFrameworkCore.SqlServer` 使用SqlServer作为EF Core数据库
> * `AutoMapper.Extensions.ExpressionMapping` 使用实体仓储并通过AutoMapper映射至数据对象
> * `AutoMapper.Extensions.Microsoft.DependencyInjection` 使用实体仓储并通过AutoMapper映射至数据对象

### Project.ServiceB
此项目作为`ServiceB`的Web服务，一般提供HTTP与GRPC服务。  
在Startup中配置`ComBoost`与其它服务依赖。
> 项目引用
> * `Project.ServiceB.Domain`
> 
> 包引用
> * `Wodsoft.ComBoost.Mvc` 使用Asp.Net Core Mvc上下文作为领域上下文
> * `Wodsoft.ComBoost.Grpc.AspNetCore` 对其它项目提供基于Grpc的领域服务
> * `Wodsoft.ComBoost.Grpc.Client` 使用其它项目提供的基于Grpc的领域服务
> * `Wodsoft.ComBoost.Distributed.RabbitMQ` 使用RabbitMQ作为分布式领域事件提供器
> * `Microsoft.EntityFrameworkCore` 使用EF Core用于实体仓储
> * `Microsoft.EntityFrameworkCore.Tools` 使用EF Core用于实体仓储

### Project.Testing
此项目用于进行单元测试、领域服务集成测试等。  
非常推荐在单元测试项目中进行领域服务的测试，领域服务单元测试一般能覆盖绝大多数功能范围。
> 项目引用
> * `Project.ServiceA.Domain`
> * `Project.ServiceB.Domain`
> 
> 包引用
> * `Wodsoft.ComBoost.Mock` 领域测试环境模拟
> * `Microsoft.EntityFrameworkCore.Sqlite` 使用Sqlite作为EF Core数据库
> * `xunit` 单元测试
> * `xunit.runner.visualstudio` 单元测试
> * `Microsoft.NET.Test.Sdk` 单元测试

### Project.DataTools
此项目一般用于提供一个命令行或UI工具，通过工具调用项目Api进行数据操作。  
工具类项目一般根据需求选择性创建。
> 项目引用
> * `Project.Client`