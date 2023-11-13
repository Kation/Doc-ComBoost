# 分布式领域服务事件
`ComBoost`提供了方便使用的分布式领域服务事件，只需要在启动配置里配置事件，然后为领域服务事件添加一些接口即可使用。

## 创建分布式领域服务事件
`ComBoost`提供了一些接口，在[`领域服务事件`](/domain/serviceevent)类型里，添加需要的接口即可用于分布式领域服务事件。

以下是`ComBoost`默认提供的分布式领域服务事件接口。

| 接口 | 事件类型 | 说明 |
| --- | --- | --- |
| IDomainDistributedHandleOnceEvent | ONCE | 事件只处理一次。 |
| IDomainDistributedMustHandleEvent | MUST | 事件必须处理。 |
| IDomainDistributedGroupEvent | GROUP | 事件根据分组进行处理，该接口实现IDomainDistributedHandleOnceEvent。 |
| IDomainDistributedDelayEvent | DELAY | 事件延迟处理。 |
| IDomainDistributedRetryEvent | RETRY | 事件处理失败后自动进行重试，事件类需要添加`DomainDistributedEventRetryTimesAttribute`特性以定义重试次数与等待时间。 |
| IDomainDistributedSingleHandlerEvent | SINGLE | 事件只通过单一处理器处理，该接口实现IDomainDistributedHandleOnceEvent。 |

`ComBoost`会对分布式领域服务事件类型进行反射获取接口，并收集接口的`DomainDistributedEventFeatureAttribute`特性反馈给分布式事件提供器。
分布式事件提供器会对收集到的特性进行分析，并不是所有的**特性或特性组合**都能支持。
也可以自行定义自己的分布式领域服务事件接口，并实现自己的分布式事件提供器。

## 配置分布式事件
项目需要添加Nuget包[`Wodsoft.ComBoost.Distributed`](https://www.nuget.org/packages/Wodsoft.ComBoost.Distributed)。
```ps
Install-Package Wodsoft.ComBoost.Distributed
```
在配置了`AddComBoost`后使用`AddDistributed`方法配置分布式服务，传入方法委托`Action<IComBoostDistributedBuilder>`并通过委托参数继续配置。
```csharp
services.AddComBoost()
    .AddDistributed(builder =>
    {
        //配置
    });
```

### RabbitMQ分布式事件提供器
项目需要添加Nuget包[`Wodsoft.ComBoost.Distributed.RabbitMQ`](https://www.nuget.org/packages/Wodsoft.ComBoost.Distributed.RabbitMQ)。
```ps
Install-Package Wodsoft.ComBoost.Distributed.RabbitMQ
```
使用`UseRabbitMQ`方法添加RabbitMQ分布式事件提供器，传递RabbitMQ连接字符串，返回`IComBoostDistributedEventProviderBuilder`接口对象。
```csharp
services.AddComBoost()
    .AddDistributed(builder =>
    {
        builder.UseRabbitMQ("RabbitMQ连接串");
    });
```
或方法委托`Action<DomainRabbitMQOptions>`对RabbitMQ进行设置选项，返回`IComBoostDistributedEventProviderBuilder`接口对象。
```csharp
services.AddComBoost()
    .AddDistributed(builder =>
    {
        builder.UseRabbitMQ(options =>
        {
            options.ConnectionString = "RabbitMQ连接串";
        });
    });
```
RabbitMQ的完整选项。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| ConnectionString | string | RabbitMQ连接字符串，优先等级最高。 |
| HostName | string | 主机名。 |
| UserName | string | 用户名。 |
| Password | string | 密码。 |
| VirtualHost | string | 虚拟主机。 |
| Port | int? | 端口，不为0时则为该指定端口。 |
| Prefix | string | 队列名称前缀。 |
| PrefetchCount | ushort | 并发处理数。 |
| UseQuorum | bool | 是否使用Quorum，默认为true。使用3.10之前的版本，请修改为false。 |
| UseDelayedMessagePlugin | bool | 是否使用延迟消息插件，默认为false。 |
| FactoryConfigure | Action\<IConnectionFactory\> | 配置连接工厂方法委托。 |
| IsCriticalHealthState | bool | RabbitMQ连接出现问题时，健康状态是否为严重，默认为true。 |

RabbitMQ支持的事件类型与行为。  

| 事件类型 | 行为 |
| --- | --- |
| MUST | 单一队列，多个消费者。事件类型必须包含**MUST**，以下省略**MUST**。 |
| ONCE | 单一队列，只消费一次。 |
| GROUP & ONCE | 有交换器，每个分组一个队列，每个分组只消费一次。 |
| DELAY | 启用延迟插件时，有延迟交换器，单一队列，延迟到期时分发至该队列。停用延迟插件时，有一个交换器，两个队列，其中一个为死信队列，TTL到期时转发至交换器，交换器分发至正常队列，需要注意的是，此时死信队列先进先出，后进消息即使超时也不会马上消费。 |
| DELAY & ONCE | 在DELAY的基础上，消息只消费一次。 |
| RETRY & ONCE | 启用延迟插件时，有延迟交换器，单一队列，延迟到期时分发至该队列。停用延迟插件时，有一个交换器，一个正常队列，每个超时时间一个死信队列，TTL到期时转发至交换器，交换器分发至正常队列。 |
| SINGLE & ONCE | 单一队列，SAC模式，只消费一次。 |
| GROUP & ONCE & SINGLE | 在GROUP&ONCE的基础上，每个分组队列都启用SAC模式。 |
| GROUP & ONCE & DELAY | 在GROUP&ONCE的基础上，交换器为延迟交换器，延迟到期时分发至各个队列。 |
| GROUP & ONCE & DELAY & SINGLE | 在GROUP&ONCE&DELAY的基础上，每个分组队列都启用SAC模式。 |

### CAP分布式事件提供器
项目需要添加Nuget包[`Wodsoft.ComBoost.Distributed.CAP`](https://www.nuget.org/packages/Wodsoft.ComBoost.Distributed.CAP)。
```ps
Install-Package Wodsoft.ComBoost.Distributed.CAP
```
使用`UseCAP`方法添加CAP分布式事件提供器，传递方法委托`Action<CapOptions>`对CAP进行设置选项，返回`IComBoostDistributedEventProviderBuilder`接口对象。
CAP选项请浏览[官方文档](https://cap.dotnetcore.xyz/user-guide/zh/cap/configuration/)。
```csharp
services.AddComBoost()
    .AddDistributed(builder =>
    {
        builder.UseCAP(options =>
        {
            //CAP配置
        });
    });
```

:::warning
###### :warning:警告
CAP仅支持**GROUP&ONCE**事件类型。
:::

### 发布分布式领域服务事件
对返回`IComBoostDistributedEventProviderBuilder`接口对象调用`AddDistributedEventPublisher`泛型方法，该泛型参数类型的领域服务事件在引发时将会使用此事件提供器进行发布。
```csharp
services.AddComBoost()
    .AddDistributed(builder =>
    {
        builder.UseXXX(options =>
        {
        }).AddDistributedEventPublisher<YourDomainServiceEventArgs>();
    });
```

### 处理分布式领域服务事件
对返回`IComBoostDistributedEventProviderBuilder`接口对象调用`AddDistributedEventHandler<THandler, TArgs>`泛型方法。
`THandler`必须实现`IDomainServiceEventHandler<TArgs>`接口，且拥有无参构造函数。
`TArgs`必须继承`DomainServiceEventArgs`。
```csharp
public interface IComBoostDistributedEventProviderBuilder
{
    IComBoostDistributedEventProviderBuilder AddDistributedEventHandler<THandler, TArgs>()
        where THandler : IDomainServiceEventHandler<TArgs>, new()
        where TArgs : DomainServiceEventArgs;
}

services.AddComBoost()
    .AddDistributed(builder =>
    {
        builder.UseXXX(options =>
        {
        }).AddDistributedEventHandler<YourEventHandler, YourDomainServiceEventArgs>();
    });
```
也可以直接传递方法委托`DomainServiceEventHandler<TArgs>`。
```csharp
public interface IComBoostDistributedEventProviderBuilder
{
    IComBoostDistributedEventProviderBuilder AddDistributedEventHandler<TArgs>(DomainServiceEventHandler<TArgs> handler)
        where TArgs : DomainServiceEventArgs;
}

services.AddComBoost()
    .AddDistributed(builder =>
    {
        builder.UseXXX(options =>
        {
        }).AddDistributedEventHandler<YourDomainServiceEventArgs>((context, e) => Task.CompletedTask);
    });
```

:::warning
###### :warning:警告
同一个事件可以添加多个处理器或委托，但是它们会同一时间执行，请注意并发操作带来的问题。
:::

### 配置分组
对返回`IComBoostDistributedEventProviderBuilder`接口对象调用`WithGroupName`方法并传入组名，可以修改此分布式事件提供器所属的组。
默认值为程序入口点程序集的名称。
```csharp
services.AddComBoost()
    .AddDistributed(builder =>
    {
        builder.UseXXX(options =>
        {
        }).WithGroupName("group1");
    });
```

:::info
###### :information_source:备注
可以为不同的事件提供器配置不同的分组。
:::

## 多个事件提供器
`ComBoost`支持在同一个项目中使用多个事件提供器，为复杂服务环境提供支持。

```csharp
services.AddComBoost()
    .AddDistributed(builder =>
    {
        builder.UseRabbitMQ("RabbitMQ服务1")
            .AddDistributedEventHandler<YourEventHandler, YourDomainServiceEventArgs1>();
            
        builder.UseRabbitMQ("RabbitMQ服务2")
            .AddDistributedEventHandler<YourEventHandler, YourDomainServiceEventArgs1>()
            .AddDistributedEventHandler<YourEventHandler, YourDomainServiceEventArgs2>()
            .AddDistributedEventPublisher<YourDomainServiceEventArgs2>();
    });
```