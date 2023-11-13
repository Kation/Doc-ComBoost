# 领域服务模板
在`ComBoost`当中，`领域服务`并不能直接调用，需要定义一个模板，来明确调用参数。  
在需要调用`领域服务`时，通过注入`领域服务模板`来调用方法。

## 定义
`领域服务模板`属于接口，需要实现`IDomainTemplate`接口，并在接口内定义调用方法与参数。

方法名称与返回类型需要与对应的`领域服务`内的方法名一致。
不同于`领域服务`的是，方法可以进行重载。
方法参数可以与`领域服务`内的方法参数不一致，但是同名参数其类型必须保持一致，不然调用时会抛出异常。
```csharp
public interface IGreeterService : IDomainTemplate
{
    Task Hello(string text);
    Task Hello(string text, string userId);
}
```
## 关联领域服务
由于`领域服务模板`是一个接口，并且其方法可以重载，参数与`领域服务`不同，所以`领域服务`不能直接实现`领域服务模板`接口。
那么就需要一些方法来将`领域服务模板`与`领域服务`关联起来。

### 使用特性声明
`ComBoost`提供了`DomainTemplateImplementerAttribute`特性，用于`领域服务`类使用。
```csharp
[DomainTemplateImplementer(typeof(IGreeterService))]
public class GreeterService : DomainService
{
    public Task Hello([FromService] ILogger<GreeterService> logger, [FromValue] string text)
    {
        logger.LogInformation($"Client say {text}");
    }
}
```
当增加了特性之后，注册领域服务时便会自动的注入特性里的`领域服务模板`接口并关联此`领域服务`。  
在相关上下文中，可以通过`IServiceProvider`获取`领域服务模板`接口并调用，例如在MVC控制器当中：
```csharp
public class GreeterController : Controller
{
    public Task Create(string text)
    {
        var service = HttpContext.RequestServices.GetRequiredService<IGreeterService>();
        return service.Hello(text);
    }
}
```

### 注册时手动关联
在配置依赖注入时，调用`AddService`添加指定的`领域服务`后可以继续通过调用`UseTemplate`方法关联`领域服务模板`接口。
```csharp
services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<GreeterService>()
            .UseTemplate<IGreeterService>();
    });
```
手动关联通常用于**泛型**`领域服务`与**泛型**`领域服务接口`。