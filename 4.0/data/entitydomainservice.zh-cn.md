# 实体领域服务
`ComBoost`提供了实体领域服务以快速使用领域服务进行增删查改操作。

## 配置
实体领域服务是一个泛型领域服务，`ComBoost`提供了扩展方法`AddEntityService<TEntity, TDto>`以快速添加领域服务。
`TEntity`泛型参数为实体类，`TDto`为该实体类对应的模型类。

```csharp
services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddEntityService<YourEntity, YourEntityModel>();
    });
```

:::info
###### :information_source:备注
同时有重载方法`AddEntityService<TEntity, TListDto, TCreateDto, TEditDto, TRemoveDto>`以在增删查改时使用不同的模型类。

使用时通过依赖注入获取领域服务模板`IEntityDomainTemplate<TListDto, TCreateDto, TEditDto, TRemoveDto>`。
如果使用扩展方法`AddEntityService<TEntity, TDto>`时还可以通过依赖注入获取领域服务模板`IEntityDomainTemplate<TDto>`，
`IEntityDomainTemplate<TDto>`实现`IEntityDomainTemplate<TDto,TDto,TDto,TDto>`。
:::

实体领域服务依赖于`AutoMapper`组件，使用该组件进行实体与模型之间的转换，所以还需要对`AutoMapper`进行配置。

```csharp
services.AddAutoMapper(config =>
{
    config.CreateMap<YourEntity, YourEntityModel>().ReverseMap();
});
```

或者使用不同的模型类时。
```csharp
services.AddAutoMapper(config =>
{
    config.CreateMap<YourEntity, YourListModel>();
    config.CreateMap<YourCreateModel, YourEntity>();
    config.CreateMap<YourEditModel, YourEntity>();
    config.CreateMap<YourRemoveModel, YourEntity>();
});
```

## 列表
`IEntityDomainTemplate<TListDto, TCreateDto, TEditDto, TRemoveDto>`接口提供了方法`Task<IViewModel<TListDto>> List()`，
以及重载方法`Task<IViewModel<TListDto>> List(int page, int size);`。

调用该方法将从数据库读取数据，并通过`AutoMapper`映射为`TListDto`模型。返回视图模型`IViewModel<TListDto>`。

:::info
###### :information_source:备注
默认情况下，列表操作不包含分页查询，如需分页查询，请添加`EntityPagerFilterAttribute`过滤器或自行通过领域服务事件实现。
:::

### 列表的领域服务事件
领域服务方法会引发以下事件。

#### EntityQueryEventArgs\<TEntity\>
事件在查询前引发，提供`IQueryable<TEntity> Queryable { get; set; }`属性供开发者修改查询条件，以及`IQueryable<T> OriginQueryable { get; }`属性拿到原始查询体。

示例：
```csharp
DomainServiceEventHandler<EntityQueryEventArgs<TEntity>> handler = (context, e) =>
{
    e.Queryable = e.Queryable.Where(t => t.IsEnabled);
};
```

#### EntityQueryModelCreatedEventArgs\<TListDto\>
事件在查询后引发，提供`ViewModel<TListDto> Model { get; }`属性以修改视图模型。

## 创建
`IEntityDomainTemplate<TListDto, TCreateDto, TEditDto, TRemoveDto>`接口提供了方法`Task<IUpdateModel<TListDto>> Create(TCreateDTO dto)`，
调用该方法将返回更新模型`IUpdateModel<TListDto>`。

接口也提供了方法`Task<IUpdateRangeModel<TListDto>> CreateRange(TCreateDTO[] dtos)`以进行批量创建。

调用该方法，将通过`AutoMapper`把`TCreateDto`模型映射为`TEntity`实体，然后通过`IEntityContext<TEntity>`插入数据库。
返回更新模型`IUpdateRangeModel<TListDto>`。

:::info
###### :information_source:备注
插入数据库前会调用`IEntity.OnCreateCompleted()`接口属性方法。
:::

### 创建的领域服务事件
领域服务方法会引发以下事件。

#### EntityMappedEventArgs\<TEntity, TCreateDTO\>
事件在模型映射为实体后引发，提供`TEntity Entity { get; }`以及`TCreateDTO Dto { get; }`属性。

#### EntityPreCreateEventArgs\<TEntity\>
事件在实体插入数据库前引发，提供`TEntity Entity { get; }`属性。

#### EntityCreatedEventArgs\<TEntity\>
事件在实体插入数据库后引发，提供`TEntity Entity { get; }`属性。

## 编辑
`IEntityDomainTemplate<TListDto, TCreateDto, TEditDto, TRemoveDto>`接口提供了方法`Task<IUpdateModel<TListDto>> Edit(TEditDTO dto)`，
调用该方法将返回更新模型`IUpdateModel<TListDto>`。

接口也提供了方法`Task<IUpdateRangeModel<TListDto>> EditRange(TEditDTO[] dtos)`以进行批量编辑。

调用该方法，将通过`AutoMapper`把`TEditDto`模型映射为`TEntity`实体（临时）。
根据映射后的`TEntity`实体（临时）的主键，使用`IEntityContext<TEntity>`从数据库获取实体（数据库）。
再通过`AutoMapper`把`TEditDto`模型映射到`TEntity`实体（数据库），然后通过`IEntityContext<TEntity>`更新该实体（数据库）进入数据库。
返回更新模型`IUpdateRangeModel<TListDto>`。

:::info
###### :information_source:备注
编辑实体前会判断`IEntity.IsEditAllowed`接口属性，为`false`时将不会更新成功。

插入数据库前会调用`IEntity.OnEditCompleted()`接口属性方法。
:::

:::warning
###### :warning:警告
编辑操作为全量字段更新，请注意并发操作引起的数据问题。
:::

### 编辑的领域服务事件
领域服务方法会引发以下事件。

#### EntityPreMapEventArgs\<TEntity, TEditDTO\>
事件在模型映射为实体前引发，提供`TEntity Entity { get; }`以及`TEditDTO Dto { get; }`属性。

#### EntityMappedEventArgs\<TEntity, TEditDTO\>
事件在模型映射为实体后引发，提供`TEntity Entity { get; }`以及`TEditDTO Dto { get; }`属性。

#### EntityPreEditEventArgs\<TEntity\>
事件在实体更新数据库前引发，提供`TEntity Entity { get; }`属性。

#### EntityEditedEventArgs\<TEntity\>
事件在实体更新数据库后引发，提供`TEntity Entity { get; }`属性。

## 删除
`IEntityDomainTemplate<TListDto, TCreateDto, TEditDto, TRemoveDto>`接口提供了方法`Task Remove(TRemoveDto dto)`。

接口也提供了方法`Task RemoveRange(TRemoveDto[] dtos)`以进行批量删除。

调用该方法，将通过`AutoMapper`把`TRemoveDto`模型映射为`TEntity`实体（临时）。
根据映射后的`TEntity`实体（临时）的主键，使用`IEntityContext<TEntity>`从数据库获取实体（数据库）。
然后通过`IEntityContext<TEntity>`从数据库删除该实体（数据库）。

:::info
###### :information_source:备注
删除实体前会判断`IEntity.IsRemoveAllowed`接口属性，为`false`时将不会更新成功。
:::

### 删除的领域服务事件
领域服务方法会引发以下事件。

#### EntityPreRemoveEventArgs\<TEntity\>
事件在实体从数据库删除前引发，提供`TEntity Entity { get; }`属性。

#### EntityRemovedEventArgs\<TEntity\>
事件在实体从数据库删除后引发，提供`TEntity Entity { get; }`属性。