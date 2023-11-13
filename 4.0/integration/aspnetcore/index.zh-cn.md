# ASP.NET Core
在ASP.NET Core中使用`ComBoost`框架。

`ComBoost`为ASP.NET Core准备了HTTP值提供器与身份提供器。

## 配置ASP.NET Core服务
项目需要添加Nuget包[`Wodsoft.ComBoost.AspNetCore`](https://www.nuget.org/packages/Wodsoft.ComBoost.AspNetCore)。
```ps
Install-Package Wodsoft.ComBoost.AspNetCore
```
使用`AddAspNetCore`扩展方法配置本地领域服务，可选参数为方法委托`Action<IComBoostAspNetCoreBuilder>`。
```csharp
builder.Services.AddComBoost()
    .AddAspNetCore();
```

### 自定义身份认证
默认情况下，[领域上下文的用户主体声明](/infrastructure/domaincontext#用户主体声明)来源于`HttpContext.User`。
使用`UseAuthentication`方法可以自定义来源，传入`Func<HttpContext, ClaimsPrincipal>`方法委托。

```csharp
builder.Services.AddComBoost()
    .AddAspNetCore(builder =>
    {
        builder.UseAuthentication(context => context.User);
    });
```

## 值提供器
`ComBoost`支持将QueryString、Form表单、Json数据作为值供领域服务使用。

当提交内容包含Form表单时，优先级为QueryString->Form表单->Form表单文件。

当提交内容包含Json数据时，优先级为QueryString->Json数据。

:::info
###### :information_source:备注
非`String`类型时，将通过`TypeDescriptor`获取**类型转换器**进行类型转换。
:::

## 映射领域服务模板
`ComBoost`支持通过ASP.NET CORE的HTTP请求调用领域服务模板的方法。

首先定义领域端点类，必须为**部分类**，继承`DomainEndpoint<T>`，其中`T`为领域服务模板接口类型。
需添加`DomainEndpointAttribute`特性，否则不会生效。
```charp
[DomainEndpoint]
public partial class YourEndpoint : DomainEndpoint<IYourDomainTemplate>
{
}
```

配置`AspNetCore`时使用`AddDomainEndpoint`扩展方法，将使用当前程序集的领域端点类。
```csharp
builder.Services.AddComBoost()
    .AddAspNetCore(builder =>
    {
        builder.AddDomainEndpoint();
    });
```

如果领域端点类不在当前程序集，可以传入`Assembly`类型参数，添加指定程序集内的领域端点类。
```csharp
builder.Services.AddComBoost()
    .AddAspNetCore(builder =>
    {
        builder.AddDomainEndpoint(typeof(YourEndpoint).Assembly);
    });
```

### 请求行为关系
`ComBoost`将通过领域服务模板方法名称，推断对应的HTTP请求行为。

| 行为 | 名称 |
| --- | --- |
| GET | 以`Get`开头的方法名 |
| PUT | 以`Update`、`Edit`开头的方法名 |
| DELETE | 以`Delete`、`Remove`开头的方法名 |
| POST | 除以上的其它方法名 |

### 参数映射关系
当行为不为`GET`时，将通过调用`GetBodyValueAsync<TValue>`方法获取参数值，并且参数数量不能超过一个。  
默认通过`System.Text.Json.JsonSerializer`进行反序列化，可以重写`GetBodyValueAsync<TValue>`方法实现自定义取值。

当行为为`GET`时，将判断参数类型调用不同的方法获取参数值。  
数组类型时，调用`TValue[] GetQueryArrayValue<TValue>`方法获取参数值。  
`List<T>`类型时，调用`List<TValue> GetQueryListValue<TValue>`方法获取参数值。  
可空类型时，调用`TValue? GetQueryNullableValue<TValue>`方法获取参数值。  
其它类型，调用`TValue? GetQueryValue<TValue>`方法获取参数值。

以上方法都可以重写以实现自定义取值。

:::info
###### :information_source:备注
参数类型不为`String`时，默认使用`TypeDescriptor`获取**类型转换器**进行类型转换。
:::

### 路由模板
`ComBoost`使用`ASP.NET CORE 路由`功能进行管道处理。  
通过`EndpointTemplate`属性获取路由模板，默认模板为`api/TemplateName/{method}`。
其中`TemplateName`为方法`GetTemplateName()`的调用结果，默认返回方法名，并去除`Endpoint`后缀。

:::warning
###### :warning:警告
重写`EndpointTemplate`属性修改路由模板时，必须保留参数`{method}`。
:::

### 结果处理
领域服务方法调用成功后，会调用`OnDomainExecuted`方法以处理返回结果。

默认使用`System.Text.Json.JsonSerializer`对返回对象进行序列化并写入HTTP响应内容。  
当返回对象为`Stream`类型时，直接写入HTTP响应内容。

重写此方法以实现自定义结果处理。

### 异常处理
领域服务方法调用出现异常时，会调用`OnDomainException`方法以处理异常内容。

默认会记录日志以及设置HTTP响应状态码500。

重写此方法以实现自定义异常处理。

### 配置映射领域服务模板端点
使用映射领域服务需要在ASP.NET Core管道内添加路由端点。
```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapDomainEndpoint();
});
```

### API说明
`ASP.NET Core 3.1`或以上版本时，支持生成`ApiDescriptor`以供其它服务生成文档，比如`Swagger`。

`ComBoost`通过`DomainEndpoint`的`GetApiDescriptions`方法获取该端点的API说明。
`DomainEndpoint`默认实现了该方法。


:::info
###### :information_source:备注
如果开发者重载了[结果处理](#结果处理)方法，则需要重载`GetSupportedResponse`方法，返回对应领域服务模板方法的返回内容说明。
如果开发者重载了`GetBodyValueAsync<TValue>`方法，则需要重载`GetSupportedRequestFormat`方法，返回对应领域服务模板方法的请求内容说明。
:::