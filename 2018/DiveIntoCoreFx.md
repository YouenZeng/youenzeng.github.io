# DotNet Framework 代码阅读

Dotnet Core最近发布了2.1版本，相信也更加趋于稳定了。 虽然我接触.NET是从3.5+开始，不过这次可以从1.0开始见证它的发展了。 最近闲的时候阅读了一部分源码，虽然之前也看过不少，不过没有记录。 现在有了博客，就记录一点。

主要看的是下面几个仓库里面的内容：

* ReferenceResource : [https://github.com/Microsoft/referencesource](https://github.com/Microsoft/referencesource)

* CoreFx : [https://github.com/dotnet/corefx](https://github.com/dotnet/corefx)

* CoreCLR: [https://github.com/dotnet/coreclr](https://github.com/dotnet/coreclr)

## Void 是个结构体（Struct)
```
Public struct Void{}
```

## AggregateException

AggregateException 是伴随着async/await引入的新的类型, 有个`Flatten`方法, 可以将内部异常展开（貌似是BFS算法）. 在调用时，将内部异常保存在一个`ReadOnlyCollection`中,不可改变. --> See how stack trace generated


## DBNull

DbNull.Value is an instance of DbNull type. ToString() => string.Empty
Inheriated from Iconvertible, all other convertation will throw exception. InvalidCastException

## DateTime

使用大量的缓存
* Days per 100 years/400years 
* 里面存储了dates to 1601/1899/1970/10000，因为对应了不同的纪元 -> https://en.wikipedia.org/wiki/Epoch_(reference_date)
* Ticks per ms/s/Minute/Hour/Day
* s_daysToMonth365/s_daysToMonth366


## 字典

Dictionay  - hash with chaining
HashTable - hash using open addressing


## Overflow检测

在数字一部分， 有一段很精妙的overflow检测方式， 在做算法题的时候会用到。
```
        public static int Abs(int value)
        {
            if (value < 0)
            {
                value = -value;
                if (value < 0)
                {
                    ThrowAbsOverflow();
                }
            }
            return value;
        }

```

以及几个有意思的常量， 对于自己设计框架有帮助。

```
        public const double NegativeInfinity = (double)-1.0 / (double)(0.0);
        public const double PositiveInfinity = (double)1.0 / (double)(0.0);
        public const double NaN = (double)0.0 / (double)0.0;

```

## Span

```
Span<T> https://msdn.microsoft.com/en-us/magazine/mt814808.aspx
```

##

在设计框架里面的类型时， 可以使用`DebuggerTypeProxy`在debugger里面显示的内容。
