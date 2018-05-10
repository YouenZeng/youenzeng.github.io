# 炉石传说套牌代码分析

## 代码

* https://github.com/HearthSim/HearthDb
* Build时候会去拖这个仓库: https://github.com/HearthSim/hsdata

https://developers.google.com/protocol-buffers/docs/encoding

http://asciiflow.com/
https://github.com/lamperi/hearthstone-deck-format/blob/master/README.md
```

AAECAQcCrwSR^AIOHLACkQP/A44FqAXUBaQG7gbnB+8HgrACiLACub8CAA==

byte[43] {
0,                +---------^   placeholder
1,                |--------->   ^ersion always 1
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