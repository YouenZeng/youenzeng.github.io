# Core FX阅读

Dotnet Core最近发布了2.1版本

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

```
Plenty of cache, Days per 100 years/400years 
dates to 1601/1899/1970/10000  -> https://en.wikipedia.org/wiki/Epoch_(reference_date)
Ticks per ms/s/Minute/Hour/Day
s_daysToMonth365/s_daysToMonth366

```

```
Dictionay  - hash with chaining
HashTable - hash open addressing

```

```
Overflow checking
--

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

```
        public const double NegativeInfinity = (double)-1.0 / (double)(0.0);
        public const double PositiveInfinity = (double)1.0 / (double)(0.0);
        public const double NaN = (double)0.0 / (double)0.0;

```
```
Span<T> https://msdn.microsoft.com/en-us/magazine/mt814808.aspx
```
```
DebuggerTypeProxy
Ew 
```