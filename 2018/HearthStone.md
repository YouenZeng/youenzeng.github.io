# 炉石传说套牌代码分析

炉石传说可以共享套牌了，复制一串代码在炉石里面就可以直接导入，有了系统的支持，导入套牌就方便多了。那么这串字符是如何生成的呢? 经过一些分析和学习，记录如下。

我在HearthSim这个网站上找到了相关的分析和代码，[https://hearthsim.info/docs/deckstrings/](https://hearthsim.info/docs/deckstrings/) 。这个网站提供了几个编程语言的生成代码，以C#版本的为例做分析。

## 代码

仓库地址： [https://github.com/HearthSim/HearthDb](https://github.com/HearthSim/HearthDb)。拖下来编译时，它会再去获取另外一个炉石数据库仓库: [https://github.com/HearthSim/hsdata](https://github.com/HearthSim/hsdata)

有了这2个仓库后， 代码就可以正常运行了。在接触陌生的代码时，了解其功能除了读文档外，就是阅读测试代码了。
PS:里面有俩项目Build会失败，组件引用冲突错乱，无视即可。只需Build `HearthDB`。

通过阅读测试方法`TestDeckStrings`，可以对这段字符有个大概了解。

```
[TestMethod]
public void TestDeckStrings()
{
    var deck = DeckSerializer.Deserialize(DeckString);
    Assert.AreEqual(CardIds.Collectible.Warrior.GarroshHellscream, deck.GetHero().Id);
    var cards = deck.GetCards();
    Assert.AreEqual(30, cards.Values.Sum());
    var heroicStroke = cards.FirstOrDefault(c => c.Key.Id == CardIds.Collectible.Warrior.HeroicStrike);
    Assert.IsNotNull(heroicStroke);
    Assert.AreEqual(2, heroicStroke.Value);
}
``` 
可以看到字符解析为byte数组，然后对其顺序遍历解析，具体分析见下图。

```
AAECAQcCrwSRvAIOHLACkQP/A44FqAXUBaQG7gbnB+8HgrACiLACub8CAA==

byte[43] {
0,                +--------->   placeholder
1,                |--------->   version always 1
2,                |--------->   FormatType
1,                |--------->   Num Heroes + always 1
7,                |--------->   HeroId
2,                +--------->   numSingleCards
175, 4,         + |
145, 188, 2,    + +-------------> single card part
14,               +---------^   numDoubleCards
28,           +
176, 2,       |
145, 3,       |
255, 3,       |
142, 5,       |
168, 5,       |
212, 5,       |
164, 6,       |  +----------->  double cards part
238, 6,       |
231, 7,       |
239, 7,       |
130, 176, 2,  |
136, 176, 2,  |
185, 191, 2,  +
0                 +----------->  multi cards (more than 2) count
}
```

PS: 我是用了ASCII Flow，效果看起来还不错. [http://asciiflow.com/](http://asciiflow.com/)

## 数字的编码：Base 128 Varints

值得一提的是里面的卡牌编号是由1~3个byte推算出来的，而不是每个占用4个byte。这样大幅度缩减了字符串的长度，尤其是新版本卡牌宇宙卡组：） 。这个数字是如何生成的呢？我们可以在`VarInt`这个类里面找到。

### 生成
```
while(value != 0)
{
    var b = value & 0x7f;
    value >>= 7;
    if(value != 0)
        b |= 0x80;
    ms.WriteByte((byte)b);
}
```
### 解析
```
ulong result = 0;
foreach(var b in bytes)
{
    var value = (ulong)b & 0x7f;
    result |= value << length * 7;
    if((b & 0x80) != 0x80)
        break;
}
```

乍一看，有点难懂，不难看出这个是128进制的表示方法，不过与我们常见的进制转换算法略有不同，这里使用了`|= 0x80`记录了一个最高有效位(most significant bit (msb) )来标识一个数字是否结束。这个自解释的信息，使得连续数字不需要其他分隔符，因为当我们读到没有MSB时，就知道这个数字标识到头了，下一个读到的新的数字的部分。这种表示方法，是倒序表示的，即低位在前面，随后读到高位然后加和得到结果。

Protocal buffers中使用了这种编码方式：[https://developers.google.com/protocol-buffers/docs/encoding](https://developers.google.com/protocol-buffers/docs/encoding)
