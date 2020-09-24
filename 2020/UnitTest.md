# 测试类型

测试代码是我们的好朋友(测试人员也是). 它可以使我们写代码的时候更自信, 不必过分担心不小心影响到其他功能. 测试可以帮助我们.

一些常规开发过程中的:
1. End to end test
1. Integration test
1. Unit test

和一些发布后进行的:
1. Smoke/Sanity test
1. Regression test

以及一些性能测试.

# 测试什么

从后端程序员的视角来看, 日常开发更关注于Unit test和Integration  test. 通过一些测试组件, 覆盖各种条件. 

# 单元测试

单元测试主要测试代码片段, 通常是方法级别的测试, 测试各种条件的输入下, 方法可以得到意料之类的结果. 测试覆盖率应该尽可能的高, 边界条件要尽可能覆盖, 这是比较理想的情况..
写可以测试的代码, 代码逻辑混乱, 方法体过大, 参数过多都会让测试变得困难. 所以测试的另一个好处就是让代码结构合理一些, 因为这些代码要可以测试, 会自然而然的被"优化".


所有的测试都可以被抽象为下面的模型.

3A: 
1. Arrange , 安排起来.. // var testObject = new TestObject();
1. Act , 执行           // var result = testObject.MethodA();
1. Assert , 断言        // Assert.AreEqual(true, result);

# 集成测试

集中在测试后端接口, 构造一个client, 发起请求, 验证API的相应状态与结果, 以及一些中间数据的状态. 相对来说每个步骤都会复杂一些, 虽然也是上面的3A模型.

//var server = new TestServer();
//var client = server.CreateClient();

//var response = await client.GetAsync("...");

//Assert.AreEqual(200, response.StatusCode);
//Assert.AreEqual("testContent", response.Content)


对于不同版本的aspnet core版本, 实现方式略有不同, 不过方式都是为了创建In-Memory server, 然后访问接口了.

## 集成测试 aspnet core 1.1

可以使用`TestServer` class来实现,`TEntryPoint`为`Startup`后者子类. 定制不太方便, 最近再研究下怎么控制.

```CSharp
            _server = new TestServer(new WebHostBuilder()
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<TEntryPoint>());
            _client = _server.CreateClient(); //get client
```

## 集成测试 aspnet core 2.2+

MSDN有详细的文档, 不做胶水程序员了! 值得注意的是开放了`WebApplicationFactory.ConfigureWebHost`, 可以方便的定制services.

```CSharp
            _factory = new CustomWebApplicationFactory<Startup>();
            _factory.WithWebHostBuilder(builder =>
            {
                var mockMailService = A.Fake<IMailService>();
                A.CallTo(() => mockMailService.SendActivationEmail(A<User>.Ignored)).Returns(Task.CompletedTask);
                builder.ConfigureServices(services =>
                {
                    services.Replace(new ServiceDescriptor(typeof(IMailService), sp => mockMailService,
                        ServiceLifetime.Scoped));
                });
            });
            
            //...
            var client = _factory.CreateClient();
```
