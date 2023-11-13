# 最新构建版本
`ComBoost`最新提交的代码不会快速发布进入`Nuget`，如需使用最新代码版本，需添加`MyGet`仓库以获取最新代码版本的包。

## 使用Directory.Build.props文件
在解决方案文件夹内，新增`Directory.Build.props`文件，并添加如下内容。

```xml
<Project>
  <PropertyGroup>
    <RestoreAdditionalProjectSources>
      https://www.myget.org/F/comboost/api/v3/index.json;
    </RestoreAdditionalProjectSources>
  </PropertyGroup>
</Project>
```

## 为Visual Studio添加程序包源

### 使用程序包管理器控制台添加

```powershell
Register-PackageSource -Name ComBoostMyGet -Location https://www.myget.org/F/comboost/api/v3/index.json -ProviderName NuGet
```

### 通过UI添加
1. 打开Visual Studio，选择 `工具` > `选项`。
2. 选择`NuGet 包管理器`，然后选择`程序包源`。
3. 输入名称`ComBoostMyGet`，源输入`https://www.myget.org/F/comboost/api/v3/index.json`，然后点击`更新`按钮。