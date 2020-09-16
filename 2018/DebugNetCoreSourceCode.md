# 调试ASP.NET Core 代码

今天在Stackoverflow上面看到一个Routing相关的问题， 看起来和想的不一样， 于是想调试下代码看看里面的细节， 结果发现非常方便，所以分享下。

我最开始的想法是从Github上面下载源码，然后Attach to Process.

先关掉`Just My Code`这个开关。

于是开始动手，本地`dotnet new webapi`，`dotnet build xx.sln`, `dotnet xx.dll`。

从github下载源码，[https://github.com/aspnet/Mvc.git](https://github.com/aspnet/Mvc.git)，打开后切换至对应版本的release分支。

![images](..\images\Snipaste_2018-10-11_23-06-39.png)

后面就直接attach 到 dotnet.exe

![images](..\images\Snipaste_2018-10-11_23-09-42.png)

此时断点是不亮的，如提示，没有加载运行所需的PDB文件: `'dotnet.exe' (CoreCLR: clrhost): Loaded 'C:\Program Files\dotnet\shared\Microsoft.AspNetCore.App\2.1.3\Microsoft.AspNetCore.dll'. Cannot find or open the PDB file.`， 我们可以去巨硬的symbol server下载， 在Visual Studio中，`Debug->Windows->Modules`,找到要调试的DLL, 右键Load Symbols即可。

![images](..\images\Snipaste_2018-10-11_23-15-04.png)。

这样就断点亮起来，就可以继续调试了。

后面发现更方便的办法就是直接在`Module`里面`Load symbols`，如果本地没有源码，VS会提示是否要下载源码，不过如果要完整调试的话，需要下载好多次的感觉，还是把代码仓库搬下来省事。

![images](..\images\Snipaste_2018-10-11_23-21-42.png)。