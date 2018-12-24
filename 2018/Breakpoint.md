# 自动触发Breakpoint

曾经有段时间在跟一个Windows service作斗争,  但是我想调试它的startUp事件代码， 没法进去。 后来找到个方法就是 `System.Diagnostics.Debugger.Launch();`，这样就可以服务在启动的时候就是提示你进入断点了。 正好最近在参与一个比较复杂且混乱的项目，利用这个可以很方便地触发想调试的代码。

```
namespace Canary
{
    class Program
    {
        static void Main(string[] args)
        {

            System.Diagnostics.Debugger.Launch();
            Console.WriteLine("Hello World!");
        }
    }
}
```



同理， JavaScript里面是用`debugger;`来触发。

附上断点的工作原理， 在hack news上看到的 [https://majantali.net/2016/10/how-breakpoints-are-set/](https://majantali.net/2016/10/how-breakpoints-are-set/)。