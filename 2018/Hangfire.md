# Hangfire使用体验

Hangfire是一个执行background job的组件，支持.Net Framework和.Net Core。 在一些中小型的项目中，如果没有专门做background job的服务的话，使用Hangfire是一个不错的选择。因为其可以直接host在WebSite中，直接跟着网站发布即可。

## 配置
配置是非常方便的，参考官方文档即可, 基于OWIN的都可以。

存储方面，Hangfire支持MSSQL, Redis, Mongo等。 我们使用的Redis的，由于Redis需要Pro版本， 可以第三方开源版本，[比如这个](https://github.com/marcoCasamento/Hangfire.Redis.StackExchange),基于StackExchange.Redis实现的，支持的.Net版本不是很全，由于代码并不复杂，不支持的版本自己编译下就好。 安装的时候会引入很多新的组件，使用的时候要留意与现有组件版本冲突。

#任务类型

Hangfire可以处理多种类型的Job。

* 即时任务
* 延时任务
* 循环任务

Pro版本支持批量的执行的一些特性。

即时任务，就是个消息队列，支持Queue的优先级，比如某个Queue优先级较高且里面有数据，则会优先把这个Queue里面的处理完成。

延时任务，就是个定时任务，本质上和上面的没什么区别，不过Hangfire把他们分开处理了，从Redis里面的数据可以看到他们的数据结构不同。

Continuations，这个功能可以多做些事情，比如失败了做一些额外的事情。

# 任务配置

Hangfire里面的任务都为Action类型，直接Enqueue时说明调用什么方法即可。 对于Generic接口的方法调用， 配置下就可以支持从IoC里面读取配置。如果IoC没有，且要用带参数的构造函数就比较惨，在ASP.NET Core里面，手动注册其实现方法。 IServiceCollection.AddTransient，应该是这个。

# 其他

* 有个基础功能的Dashboard，如果要开启远程登陆，要配置Authentication
* 支持Job的失败重试
* 记得看Deployment to PROD那一段的说明！