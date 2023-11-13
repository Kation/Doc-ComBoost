# 通用主机
`ComBoost`非常依赖于依赖注入服务，即使是控制台程序，也推荐使用`.NET通用主机`作为基础设施进行配置使用。

## 配置ComBoost
参考`.NET通用主机`的使用方法，在配置服务时进行`ComBoost`相关的配置。  

项目需要添加Nuget包[`Wodsoft.ComBoost`](https://www.nuget.org/packages/Wodsoft.ComBoost)。
```ps
Install-Package Wodsoft.ComBoost
```

`ComBoost`提供了一系列扩展方法用于`IServiceCollection`对象进行快速配置。

使用`AddComBoost`扩展方法配置领域服务事件管理器、身份认证提供器、领域上下文提供器。  
方法返回`IComBoostBuilder`接口用于继续配置其它`ComBoost`服务。
```csharp
var builder = Host.CreateDefaultBuilder(args);
builder.Services.AddComBoost();
```

## 配置本地领域服务
使用`AddLocalService`扩展方法配置本地领域服务，参数为方法委托`Action<IComBoostLocalBuilder>`。

```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {

    });
```

### 添加领域服务
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<YourService>();
    });
```

### 为领域服务指定领域模板
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<YourService>()
            .UseTemplate<YourServiceTemplate>();
    });
```

### 为领域服务设置领域服务事件处理
使用事件处理器。
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<YourService>()
            .UseEventHandler<YourEventHandler, EventType>();
    });
```
或直接使用委托。
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<YourService>()
            .UseEventHandler<EventType>((context, e) =>
            {
                return Task.CompletedTask;
            });
    });
```

### 为领域服务设置过滤器
为整个领域服务设置过滤器。
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<YourService>()
            .UseFilter<YourFilter>();
    });
```
或为指定的领域服务方法设置过滤器。
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<YourService>()
            .UseFilter<YourFilter>("Method1", "Method2", ...);
    });
```

### 添加全局领域事件处理
使用事件处理器。
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddEventHandler<YourEventHandler, EventType>();
    });
```
或直接使用委托。
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddEventHandler<EventType>((context, e) =>
        {
            return Task.CompletedTask;
        });
    });
```

### 添加程序集里的所有领域服务
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddServiceFromAssembly(typeof(YourService).Assembly);
    });
```

## 添加空领域上下文提供器
调用领域服务的来源没有上下文数据时，可以使用空领域上下文提供器。
```csharp
builder.Services.AddComBoost()
    .AddEmptyContextProvider();
```

## 清空领域上下文提供器
`ComBoost`支持多个领域上下文提供器，并按顺序进行配置。
可以将之前配置的领域上下文提供器清空，然后添加新的领域上下文提供器。
```csharp
builder.Services.AddComBoost()
    .AddSomeDomainContextProvider1()
    .AddSomeDomainContextProvider2()
    .ClearDomainContextProvider()
    .AddSomeDomainContextProvider3();
```

## 添加全局领域服务过滤器
```csharp
builder.Services.AddComBoost()
    .AddGlobalFilter<YourFilter>();
```