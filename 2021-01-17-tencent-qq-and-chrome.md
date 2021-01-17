

## [原创]关于QQ读取Chrome历史记录的澄清 



今天看到群里有同学发了一篇v2ex上的帖子[https://www.v2ex.com/t/745030](https://www.v2ex.com/t/745030)，说QQ会读取Chrome的历史记录，被火绒自定义规则拦截了，本来我是不信的，但是他说他复现了，而且是QQ登录10分钟后才会去访问。

 

这我就想去验证下了，开虚拟机装QQ、Chrome，然后打开Process Monitor开始等。规则简单的过滤下。

 
![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/1.png)
果然看到了读取AppData\Local\Google\Chrome\User Data\Default\History等目录的操作。

 

![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/2.png)
而且时间也是恰到好处的十分钟。

 

![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/6.png)
这是实锤了QQ和Chrome过不去啊，这我可不信，把规则去掉，重新翻了一下才发现果然是冤枉QQ了啊。

 

![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/4.png)
受害人之多令人震惊，仔细一看，这玩意是遍历了Appdata\Local\下的所有文件夹，然后加上User Data\Default\History去读啊。User Data\Default\History是谷歌系浏览器（火狐等浏览器不熟，不清楚目录如何）默认的历史纪录存放位置，Chrome中枪也就很正常了。

 

然后就该研究研究QQ为啥要这么干了，读取到的浏览器历史记录又拿来干啥了呢？

 

挂上x32dbg，动态调试找到位置。

 

![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/5.png)
然后去IDA里直接反编译出来，如下（位置在AppUtil.dll中 .text:510EFB98 附近）

 

![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/6.png)
这一段的逻辑还是很好看懂的，先读取各种 User Data\Default\History 文件，读到了就复制到Temp目录下的temphis.db。回去看下Procmom，果然没错。

 

![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/7.png)
再之后的操作就简单了，SQLite读取数据库，然后“select url from urls”，这是在干什么大家都懂哈。后面就不接着讲了，有兴趣的可以自己接着看。

 

结论，QQ并不是特意读取Chrome的历史记录的，而是会试图读取电脑里所有谷歌系浏览器的历史记录并提取链接，确认会中招的浏览器包括但不限于Chrome、Chromium、360极速、360安全、猎豹、2345等浏览器。

 

晚上来编辑一下，刚才去试了下TIM，果然经典重现，而且比QQ还离谱，不多说直接上图。

 

![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/8.png)
![](http://disk.seimo.pw/%E5%9B%BE%E5%BA%8A/mirror/tencent-qq-2021-01-17/9.png)
