# Core FX阅读

Dotnet Core最近发布了2.1版本

```
Public struct Void{}
```

```
AggragationException , can be flatten, merge message by (). Consist inner exceptions in a ReadOnlyCollection, every time copy a new one. --> See how stack trace generated
```

```
DbNull.Value is an instance of DbNull type. ToString() => string.Empty
Inheriated from Iconvertible, all other convertation will throw exception. InvalidCastException

```

```
DateTime. 
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