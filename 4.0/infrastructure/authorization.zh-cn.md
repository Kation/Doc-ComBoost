# 身份授权
应用程序当中，身份验证一般根据入口类型，由入口框架进行验证。比如[ASP.NET Core的身份验证](https://learn.microsoft.com/zh-cn/aspnet/core/security/authentication/?view=aspnetcore-7.0)。
领域服务上下文会从入口框架处获取得到用户主体声明。

领域服务当中一般会进行身份授权验证，简单的说就是判断用户是否拥有角色。

## 授权过滤器
`ComBoost`提供了授权过滤器特性`AuthorizeAttribute`，以方便开发者在领域服务当中使用。

构造函数传入角色数组，当用户所有角色都不拥有时抛出`UnauthorizedAccessException`异常。

当使用空构造函数时，将判断用户主体声明，如未通过验证时抛出`UnauthorizedAccessException`异常。

```csharp
[Authorize("role1", "role2")]
public class YourDomainService : DomainService
{

}
```

授权过滤器特性可以添加多个，从而达成必须拥有多个角色的授权条件。

```csharp
[Authorize("role1")]
[Authorize("role2")]
public class YourDomainService : DomainService
{

}
```

## 授权提供器
授权过滤器内部将通过依赖注入获取`IAuthorizationProvider`接口，`ComBoost`默认使用`DefaultAuthorizationProvider`类作为实现。

`IAuthorizationProvider`接口仅拥有一个方法，检查是否拥有角色，并返回拥有的角色字符串数组。

```csharp
public interface IAuthorizationProvider
{
    Task<string[]> CheckInRoles(IDomainExecutionContext context, params string[] roles);
}
```

`DefaultAuthorizationProvider`将通过领域执行上下文的领域上下文属性拿到用户主体声明，并使用用户主体声明的`IsInRole`方法判断是否包含角色，并返回拥有的角色字符串数组。

开发者可以自己实现授权提供器，并通过配置`ComBoost`时使用`AddAuthorizationProvider`方法添加自定义授权提供器。

```csharp
var builder = Host.CreateDefaultBuilder(args);
builder.Services.AddComBoost()
    .AddAuthorizationProvider<YourAuthorizationProvider>();
```

:::info
###### :information_source:备注
授权提供器的生命周期为`Scoped`。
:::

一般情况下，只有在入口框架身份验证后得到的用户主体声明不包含角色信息的情况下，才需要实现自定义授权提供器。  
并在自定义授权提供器内访问数据库获取相关角色信息进行判断。