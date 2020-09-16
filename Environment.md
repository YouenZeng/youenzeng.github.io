# aspnet website load config by environment

在产品的发布流程中，通常会分几个阶段如DEV,QA,Pre-production,production等，根据[发布环境](https://en.wikipedia.org/wiki/Deployment_environment)动态加载网站的配置文件是一个比较常见的需求。
在ASPNET和aspnetcore中，有着不同的实现方式。

## aspnet

在传统ASPNET项目中，采用`Web.config`做配置文件，这种XML结构的文件，微软提供了[Transform(转换)语法](https://msdn.microsoft.com/en-us/library/dd465318(v=vs.100).aspx)的功能。
通过这种aspnet，可以实现对XML节点的追加、替换、删除。在Web.relase.config中，就使用了这种功能来移除调试的配置节点。在编译的过程中，可以声明配置的名称，生成最终的Web.config为转换后的结果。

## aspnetcore

在aspnetcore中，可以根据当前[运行环境](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments)来决定使用哪个配置文件。这个运行环境读取自host进程的`ASPNETCORE_ENVIRONMENT`环境变量。ASPNETCORE的配置文件为appsettings.json格式，加载时采用子配置文件覆盖的策略，即appsettings.dev.json节点会覆盖appsettings.json的。

## 结论

通过上面的对比，可以看到aspnet项目是编译时确定环境，而aspnetcore是运行时确定环境，后者可以一次编译随地运行，在便捷上，aspnetcore优于传统的aspnet。
但是在现代化的DevOps工具支持下，可以通过配置分发解决大部分问题。（待研究）