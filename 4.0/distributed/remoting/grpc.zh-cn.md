# gRPC远程调用
`ComBoost`支持gRPC客户端与服务端。

## gRPC客户端
项目需要添加Nuget包[`Wodsoft.ComBoost.Grpc.Client`](https://www.nuget.org/packages/Wodsoft.ComBoost.Grpc.Client)。
```ps
Install-Package Wodsoft.ComBoost.Grpc.Client
```

### 配置gRPC服务
使用`AddGrpcService`扩展方法配置本地领域服务，参数为方法委托`Action<IComBoostGrpcBuilder>`。  
使用`AddService`方法添加一个gRPC服务地址并使用`UseTemplate`关联一个领域服务模板接口。
```csharp
builder.Services.AddComBoost()
    .AddGrpcService(builder =>
    {
        builder.AddService(new Uri("https://hostname/"))
            .UseTemplate<YourServiceTemplate>();
    });
```
不同服务可以存在于不同的地址。
```csharp
builder.Services.AddComBoost()
    .AddGrpcService(builder =>
    {
        builder.AddService(new Uri("https://serviceA/"))
            .UseTemplate<YourServiceTemplateA>();
        builder.AddService(new Uri("https://serviceB/"))
            .UseTemplate<YourServiceTemplateB>();
    });
```

配置过后，可以在依赖注入中直接获取对应的领域服务模板接口并直接调用使用。

### 身份信息透传
`ComBoost`支持将客户端上下文中的身份信息传递到服务端，在服务端的gRPC请求上下文中，将能够直接通过身份认证提供器拿到客户端的身份信息。  
通过`AddAuthenticationPassthrough`方法添加身份信息透传功能。
```csharp
builder.Services.AddComBoost()
    .AddGrpcService(builder =>
    {
        builder.AddAuthenticationPassthrough();
    });
```

## gRPC服务端
`ComBoost`基于ASP.NET Core的gRPC功能提供服务。
项目需要添加Nuget包[`Wodsoft.ComBoost.Grpc.AspNetCore`](https://www.nuget.org/packages/Wodsoft.ComBoost.Grpc.AspNetCore)。
```ps
Install-Package Wodsoft.ComBoost.Grpc.AspNetCore
```

### 配置gRPC服务
配置ASP.NET Core服务时，通过委托方法添加gRPC服务功能。
指定需要启动gRPC服务的领域服务模板接口。
```csharp
builder.Services.AddComBoost()
    .AddAspNetCore(builder =>
    {
        builder.AddGrpcServices()
            .AddTemplate<YourServiceTemplate>();
    });
```

### 配置gRPC服务端点
然后配置ASP.NET Core管道，添加领域服务gRPC端点。
```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapDomainGrpcService();
});
```

### 身份信息透传
如果客户端配置了身份信息透传，服务端也需要配置才能起作用。  
通过`AddAuthenticationPassthrough`方法添加身份信息透传功能。
```csharp
builder.Services.AddComBoost()
    .AddAspNetCore(builder =>
    {
        builder.AddGrpcServices()
            .AddAuthenticationPassthrough();
    });
```

### 自定义身份信息
默认情况下，[领域上下文的用户主体声明](/infrastructure/domaincontext#用户主体声明)来源于`HttpContext.User`。
使用`UseAuthentication`方法可以自定义来源，传入`Func<HttpContext, IDomainRpcRequest, ClaimsPrincipal>`方法委托。

```csharp
builder.Services.AddComBoost()
    .AddAspNetCore(builder =>
    {
        builder.UseAuthentication((context, request) => context.User);
    });
```

:::info
###### :information_source:备注
身份信息透传或自定义身份信息会互相覆盖，只有最后一次调用的生效。
:::

### 映射关系
`ComBoost`使用`/{templateName}/{methodName}`作为gRPC服务的入口点。  
`{templateName}`为领域服务模板接口名称。