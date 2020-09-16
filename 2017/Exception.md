# Expceiton rethrow

最近在改一个项目的bug,反射调用一段代码出错,结果在Log里面看不到完整的错误. 原因是Log里面记录的是抛出的Exception,而不是内部Exception. 当我修改为抛出内部Exception时,发现记录下来Exception的StackTrace是在抛出Exception的地方,而不是引发Exception的地方, 即Exception的StackTrace被修改了.

为了重现这个问题,添加一个dll

```
    class Processor
    {
        public string Runner(DateTime dateTime)
        {
            if (dateTime.Year > 2058)
                return "Hell! It's about time!";

            throw new Exception("Hold on..");
        }
    }
```

调用代码

```
class PluginRunner
    {
        public void ExecuteCore()
        {
            Assembly assembly = Assembly.LoadFrom("PluginTest.dll");

            object obj = assembly.CreateInstance("PluginTest.Processor");
            MethodInfo m = obj.GetType().GetMethod("Runner");

            if (m != null)
            {
                object[] methodParams = new object[] { DateTime.Now };
                var result = m.Invoke(obj, methodParams);
            }
        }
    }
```

```
        static void Main(string[] args)
        {
            PluginRunner runner = new PluginRunner();
            try
            {
                runner.ExecuteCore();
            }
            catch (Exception ex)
            {
                Debug.WriteLine($"exception stacktrace:" + Environment.NewLine + ex.StackTrace + Environment.NewLine);
                throw;
            }

        }
```

由于调用方的异常抛出,此时, 记录异常为`m.Invoke`的堆栈,结果如下:

 ```
 exception stacktrace:
   at System.RuntimeMethodHandle.InvokeMethod(Object target, Object[] arguments, Signature sig, Boolean constructor)
   at System.Reflection.RuntimeMethodInfo.UnsafeInvokeInternal(Object obj, Object[] parameters, Object[] arguments)
   at System.Reflection.RuntimeMethodInfo.Invoke(Object obj, BindingFlags invokeAttr, Binder binder, Object[] parameters, CultureInfo culture)
   at System.Reflection.MethodBase.Invoke(Object obj, Object[] parameters)
   at Canary.PluginRunner.ExecuteCore() in xxx\Canary\PluginRunner.cs:line 23
   at Canary.Program.Main(String[] args) in xxx\Canary\Program.cs:line 19
 ```

随后,尝试抛出InnerException

```
                try
                {
                    var result = m.Invoke(obj, methodParams);
                }
                catch (Exception ex)
                {
                    if (ex.InnerException != null)
                    {
                        // ExceptionDispatchInfo.Capture(ex.InnerException).Throw();
                        throw ex.InnerException;
                    }
                    throw ex;
                }
```

得到的是Invoke调用方的异常,结果如下:

```
exception stacktrace:
   at Canary.PluginRunner.ExecuteCore() in xxx\Canary\PluginRunner.cs:line 33
   at Canary.Program.Main(String[] args) in xxx\Canary\Program.cs:line 19
```

但是在断点出查看时`at PluginTest.Processor.Runner(DateTime dateTime)`,异常在抛出后被修改了.

解决办法:
第一种是使用.NET 4.5中引入的`ExceptionDispatchInfo`,如官方文档所说,这种方式会保留原始堆栈.这个class的引入应该初衷是为了配合Task调用的`AggregateException`.
`
The ExceptionDispatchInfo object stores the stack trace information and Watson information that the exception contains at the point where it is captured. The exception can be thrown at another time and possibly on another thread by calling the ExceptionDispatchInfo.Throw method. The exception is thrown as if it had flowed from the point where it was captured to the point where the Throw method is called.
`
代码如下:

```
ExceptionDispatchInfo.Capture(ex.InnerException).Throw();
```

此时异常如下,原始异常被追加上去了:

```
exception stacktrace:
   at PluginTest.Processor.Runner(DateTime dateTime)
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at Canary.PluginRunner.ExecuteCore() in xxx\Canary\PluginRunner.cs:line 32
   at Canary.Program.Main(String[] args) in xxx\Canary\Program.cs:line 19
```

SO大佬在2010年指出这是一个Windows CLR的限制:
`This is a well known limitation in the Windows version of the CLR. It uses Windows' built-in support for exception handling (SEH). Problem is, it is stack frame based and a method has only one stack frame. You can easily solve the problem by moving the inner try/catch block into another helper method, thus creating another stack frame. Another consequence of this limitation is that the JIT compiler won't inline any method that contains a try statement.`

那么Linux下呢, 我在WSL下看了看是确实没有这个问题.
```
➜  CanaryNetCore git:(master) ✗ dotnet run

Unhandled Exception: System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.Exception: Hold on..
   at PluginTestCore.Processor.Runner(DateTime dateTime)
   --- End of inner exception stack trace ---
   at System.RuntimeMethodHandle.InvokeMethod(Object target, Object[] arguments, Signature sig, Boolean constructor)
   at System.Reflection.RuntimeMethodInfo.UnsafeInvokeInternal(Object obj, Object[] parameters, Object[] arguments)
   at CanaryNetCore.PluginRunner.ExecuteCore() in /mnt/c/DevLab/Canary/CanaryNetCore/PluginRunner.cs:line 33
   at CanaryNetCore.Program.Main(String[] args) in /mnt/c/DevLab/Canary/CanaryNetCore/Program.cs:line 18
   
➜  CanaryNetCore git:(master) ✗ cat cat PluginRunner.cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Runtime.ExceptionServices;
using System.Text;
using System.Threading.Tasks;

namespace CanaryNetCore
{
    class PluginRunner
    {
        public void ExecuteCore()
        {
            Assembly assembly = Assembly.LoadFrom("PluginTestCore.dll");

            object obj = assembly.CreateInstance("PluginTestCore.Processor");
            MethodInfo m = obj.GetType().GetMethod("Runner");

            if (m != null)
            {
                object[] methodParams = new object[] { DateTime.Now };
                try
                {
                    var result = m.Invoke(obj, methodParams);
                }
                catch (Exception ex)
                {
                    //if (ex.InnerException != null)
                    //{
                    //    ExceptionDispatchInfo.Capture(ex.InnerException).Throw();
                    //}
                    throw;
                }

            }
        }
    }
}

```

第二种方法是直接throw,在外面处理InnerException, 由于我不想修改外部代码, 所以使用了第一种方法.
`The recommended way to re-throw an exception is to simply use the throw statement in C# and the Throw statement in Visual Basic without including an expression. This ensures that all call stack information is preserved when the exception is propagated to the caller.`

参考:
1. https://stackoverflow.com/questions/57383/in-c-how-can-i-rethrow-innerexception-without-losing-stack-trace
2. https://msdn.microsoft.com/en-us/library/system.runtime.exceptionservices.exceptiondispatchinfo%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396
3. https://msdn.microsoft.com/en-us/library/system.exception(v=vs.110).aspx