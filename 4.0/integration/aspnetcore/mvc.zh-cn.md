# ASP.NET Core Mvc
在ASP.NET Core Mvc中使用`ComBoost`框架。

`ComBoost`为ASP.NET Core Mvc准备了MVC值提供器。

## 配置ASP.NET Core Mvc服务
项目需要添加Nuget包[`Wodsoft.ComBoost.Mvc`](https://www.nuget.org/packages/Wodsoft.ComBoost.Mvc)。
```ps
Install-Package Wodsoft.ComBoost.Mvc
```
使用`AddMvc`扩展方法配置用于`ComBoost`的Mvc相关服务。
```csharp
builder.Services.AddComBoost()
    .AddMvc();
```

## 领域方法接口
`ComBoost`提供了`IDomainAction<T>`接口，开发者可以在自己的Mvc控制器内，实现该接口。控制器必须增加部分类`partial`关键字。

`IDomainAction<T>`是一个泛型接口，泛型参数类型必须实现`IDomainTemplate`领域模板接口。

```csharp
public partial class MyController : ControllerBase, IDomainAction<IMyDomainTemplate>
{

}
```

`ComBoost`会自动为控制器实现该领域模板接口内的方法，并根据方法名确定HTTP请求行为。

### 请求行为关系
`ComBoost`将通过领域服务模板方法名称，推断对应的HTTP请求行为。

| 行为 | 名称 |
| --- | --- |
| GET | 以`Get`开头的方法名 |
| PUT | 以`Update`、`Edit`开头的方法名 |
| DELETE | 以`Delete`、`Remove`开头的方法名 |
| POST | 除以上的其它方法名 |

:::info
###### :information_source:备注
可以通过对接口方法添加HTTP行为特性，将使用该行为特性而不是通过方法名推断HTTP行为。
:::

### 参数映射关系
当行为不为`GET`时，参数将添加`FromBodyAttribute`特性，并且参数数量不能超过一个。

当行为为`GET`时，参数将添加`FromQueryAttribute`特性。

:::info
###### :information_source:备注
参数带有`From`特性时，将使用该特性而不是通过推断行为进行添加。

参数的其它特性将一同添加至控制器内。
:::

## 实体API控制器
`ComBoost`提供了便捷的实体增删查改API控制器，开发者可以简单的通过继承类以使用控制器。

项目需要添加Nuget包[`Wodsoft.ComBoost.Mvc.Data`](https://www.nuget.org/packages/Wodsoft.ComBoost.Mvc.Data)。
```ps
Install-Package Wodsoft.ComBoost.Mvc.Data
```

基类`EntityApiController<TEntity, TListDTO, TCreateDTO, TEditDTO, TRemoveDTO>`继承`ControllerBase`，提供了列表查询、创建、批量创建、编辑、批量编辑、删除、批量删除功能。

一般而言，开发者应实现一个自己的实体API控制器基类，为控制器增加特性以在MVC当中使用，并且应重写方法，增加响应结果类型特性、返回自定义对象、改写领域服务上下文。

```csharp
[ApiController]
[Route("api/[controller]/[action]")]
public class BaseApiController<TEntity, TListDTO, TCreateDTO, TEditDTO, TRemoveDTO> : EntityApiController<TEntity, TListDTO, TCreateDTO, TEditDTO, TRemoveDTO>
    where TEntity : class, IEntity
    where TListDTO : class
    where TCreateDTO : class
    where TEditDTO : class
    where TRemoveDTO : class
{
    //重写方法以增加响应类型特性
    [ProducesResponseType(typeof(ApiRsult<IViewModel<TListDTO>>), 200)]
    public override Task<IActionResult> List()
    {
        return base.List();
    }

    //重写方法以返回自定义结果
    protected override Task<IActionResult> HandleListModelAsync(IViewModel<UserDto> viewModel)
    {
        return Task.FromResult<IActionRsult>(Ok(ApiResult.Success(viewModel)));
    }

    //更多方法重写...
}
```

### 列表查询

提供公共的`List`方法，默认为`GET`行为。
可通过重写`HandleListModelAsync`方法自定义返回内容。

### 列表数据过滤

重写`HasQueryFilter`属性返回`true`，会将`QueryFilter`方法作为查询事件处理委托，可在方法内对查询体进行操作以达到过滤数据的目的。

### 创建

提供公共的`Create`与`CreateRange`方法，默认为`POST`行为。
可通过重写`HandleCreateModelAsync`与`HandleCreateRangeModelAsync`方法自定义返回内容。

### 编辑

提供公共的`Edit`与`EditRange`方法，默认为`PUT`行为。
可通过重写`HandleEditModelAsync`与`HandleEditRangeModelAsync`方法自定义返回内容。

### 删除

提供公共的`Remove`与`RemoveRange`方法，默认为`DELETE`行为。
可通过重写`HandleRemoveModelAsync`与`HandleRemoveRangeModelAsync`方法自定义返回内容。
