# 领域服务事件
在`ComBoost`当中，`领域服务`可以引发事件，通过事件处理来扩展功能。

## 事件类型
`领域服务事件`体现为一个类，必须要继承`DomainServiceEventArgs`基类。
```csharp
public class DomainServiceEventArgs : DomainServiceEventArgs
{
    public bool IsHandled { get; set; }
}

public class GreeterEventArgs : DomainServiceEventArgs
{
}
```

## 引发事件
在`领域服务`当中，可以通过调用`RaiseEvent<TArgs>(TArgs e)`方法引发传入的事件类型事件。
方法返回`Task`类型，为异步执行，可等待。
```csharp
await RaiseEvent(new GreeterEventArgs());
```

## 事件处理
在`ComBoost`当中，事件的处理机制由`领域服务事件管理器`实现。  
事件的处理通过委托执行。
```csharp
public delegate Task DomainServiceEventHandler<T>(IDomainExecutionContext context, T e) where T : DomainServiceEventArgs;
```

### 全局范围
在配置依赖注入时，添加本地服务后，可以通过`IComBoostLocalBuilder`的`AddEventHandler`方法添加全局事件处理委托。
```csharp
public interface IComBoostLocalBuilder
{
    IComBoostLocalBuilder AddEventHandler<TArgs>(DomainServiceEventHandler<TArgs> handler)
        where TArgs : DomainServiceEventArgs;
}

builder.AddEventHandler<GreeterEventArgs>((context, e) => Task.Completed);
```
通过传入事件处理委托，在引发事件时，会调用此委托以执行相关代码。

当然也可以将事件委托代码，放在一个单独的类里，实现事件处理接口。
```csharp
public class GreeterEventHandler : IDomainServiceEventHandler<GreeterEventArgs>
{
    public async Task Handle(IDomainExecutionContext context, InRequestCompletedEventArgs args)
    {
        return Task.Completed;
    }
}
```
然后通过`IComBoostLocalBuilder`的`AddEventHandler`重载方法添加全局事件处理器。
```csharp
public interface IComBoostLocalBuilder
{
    IComBoostLocalBuilder AddEventHandler<THandler, TArgs>()
        where THandler : IDomainServiceEventHandler<TArgs>, new()
        where TArgs : DomainServiceEventArgs;
}

builder.AddEventHandler<GreeterEventHandler, GreeterEventArgs>();
```

### 领域服务范围
在配置依赖注入时，添加本地`领域服务`后，可以通过`IComBoostLocalServiceBuilder<TService>`的`UseEventHandler`方法添加事件处理委托。
```csharp
public interface IComBoostLocalServiceBuilder<TService>
{
    IComBoostLocalServiceBuilder<TService> UseEventHandler<TArgs>(DomainServiceEventHandler<TArgs> handler)
        where TArgs : DomainServiceEventArgs;
}

builder.UseEventHandler<GreeterEventArgs>((context, e) => Task.Completed);
```
通过传入事件处理委托，在该`领域服务`引发事件时，会调用此委托以执行相关代码。

和全局事件处理一样，可以放在一个单独的类里，通过重载方法添加事件处理器。
```csharp
public interface IComBoostLocalServiceBuilder<TService>
{
    IComBoostLocalServiceBuilder<TService> UseEventHandler<TArgs>(DomainServiceEventHandler<TArgs> handler)
        where TArgs : DomainServiceEventArgs;
}

builder.UseEventHandler<GreeterEventHandler, GreeterEventArgs>();
```

### 领域上下文范围
在通过依赖注入获取`领域服务模板`时，可以通过`Context`属性获取`领域上下文`。
通过`领域上下文`的`事件管理器`添加事件处理器。
```csharp
public interface IDomainServiceEventManager
{
    void AddEventHandler<T>(DomainServiceEventHandler<T> handler) where T : DomainServiceEventArgs;
}

public class GreeterController : Controller
{
    public Task Create(string text)
    {
        var service = HttpContext.RequestServices.GetRequiredService<IGreeterService>();
        service.Context.EventManager.AddEventHandler<GreeterEventArgs>((context, e) => Task.Completed);
        return service.Hello(text);
    }
}
```
`领域上下文`范围添加事件处理时，仅支持添加委托处理。

## 事件处理执行顺序
领域上下文范围->领域服务范围->全局范围  
每个范围内，最先添加的事件处理最先执行。

## 声明领域服务事件
`ComBoost`提供了`DomainServiceEventAttribute`特性用于显式声明`领域服务模板`或其中的某个方法可能引发的`领域服务事件`事件类型。
```csharp
public class GreeterService : DomainService
{
    public Task Hello([FromService] ILogger<GreeterService> logger, [FromValue] string text)
    {
        logger.LogInformation($"Client say {text}");
        await RaiseEvent(new GreeterEventArgs());
    }
}

public interface IGreeterService : IDomainTemplate
{
    [DomainServiceEvent(typeof(GreeterEventArgs))]
    Task Hello(string text);
    [DomainServiceEvent(typeof(GreeterEventArgs))]
    Task Hello(string text, string userId);
}
```
这是为了帮助别人了解此`领域服务`提供了什么事件，一般用于框架当中。