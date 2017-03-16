---
title: 关于CDN和Web优化的十一个问答
date: 2017-03-16 14:34:24
tags: [CDN,Web,网站优化,翻译,转载]
categories: Web
---

&#8195;&#8195;我一直被问到关于内容分发网络(CDNs)和它如何融入更大范围的性能优化情景中，不仅在Straneloop/Radware(作者工作的公司)内部，而且在公司外。这让我惊奇地意识到我之前还从未总结过这些最有共性的问题。我写下这些尽可能为了帮助我自己能够在未来在一个地方找到这些答案，而且你也可能会发现这篇文章很有用。

<!-- more -->

译自[ 11 questions (and answers) about content delivery networks and web performance](http://www.webperformancetoday.com/2013/06/12/11-faqs-content-delivery-networks-cdn-web-performance/)

# 1. CDN是用来解决什么性能问题的？
虽然CDN也能解决一些附加的问题如**提高全局可用性**和**降低带宽（使用）**，它针对的主要问题还是**延迟：服务器接收、执行和分发请求资源所消耗的时间**（图片，CSS文件等等）。延迟的高低很大程度取决于用户距离服务器多远，并且一个页面包含资源的数量也会有影响。  

例如，如果你的页面的所有资源都放置在旧金山，一个用户在伦敦访问你的页面那么每一个请求都将会经过很长的路由过程从伦敦到旧金山再回到伦敦。如果你的页面包含100个资源对象（这是一个很正常的情况），那么你的用户的浏览不得不发送100个独立的请求到你的服务器来获取这些对象。  

通常来说，延迟在75-140ms，但是也有可能变得很高，尤其是对移动用户通过3G浏览网页来说，延迟会很容易变多2-3秒。当你在考虑许多可能减慢你的网页的因素时这是一个大问题。  

**一个CDN缓存静资源在区域胡或世界范围内的分布式服务器中（又被叫做边界缓存，入网点或者PoPs），以此使资源距离用户更近，降低路由时间**，如下图所示：  
![figure](http://ohvmg8dgt.bkt.clouddn.com/BLOG-CDN-network.png)  
# 2. CDN在各种情况下都有用吗？
CDN对许多网站是必须的，但是并不是对所有网站。例如，如果你的网站服务器就在本地并且你的用户也是本地的，CDN就没有用。  

相反有些网站的所有者相信，CDN不是独立的性能解决方案。你需要牢记，在这个电子商务和软件即服务（两种概念）的世界，两个最普遍的性能痛点是第三方内容和服务器性能。你的CDN在这方面帮不了你。  

# 3. 所有的CDNs是对等的吗？
CDN的选择真的很关键。你的利益很多取决于你选了哪个供应商，也取决于你的CDN如何存储内容和它们的入网点距离用户多元。**为了给你的网站选择最高效率的CDN，你首先需要了解你的用户在哪。**  

同时，在不同的供应商之间，延迟会差别巨大。大约一年前，我做了一些临时测试来测量我们的三个客户的网站，排除其他因素，桌面和3G网络的延迟差异。每一个我们测量的网站用了一个不同的CDN，可以看到，在桌面延迟的中位数附近有一个相当宽的范围：  

![figure](http://ohvmg8dgt.bkt.clouddn.com/desktop-latency-1-NEW.jpg)  
![figure](http://ohvmg8dgt.bkt.clouddn.com/desktop-latency-3-NEW.jpg)  
![figure](http://ohvmg8dgt.bkt.clouddn.com/desktop-latency-2-NEW.jpg)  

# 4. CDN的性能会有差异吗，即使是同一个CDN？  
在前一段的[播客采访](http://www.webperformancetoday.com/2013/02/22/aaron-peters-turbobytes-why-all-cdns-are-not-created-equal-podcast/)中，Turbobytes的创始人Aaron Peters表示，在他们的公司学习了一些现实世界性能的一些领军CDN提供商以后，他发现**性能的波动超出预期**。（在Velocity的最后一年，Aaron发表了一个杰出的回忆在[measuring CDN performance](http://cdn.oreillystatic.com/en/assets/1/event/79/Getting%20A%20Grip%20On%20CDN%20Performance_%20Why%20And%20How%20Presentation.pdf)，如果你对这个领域感兴趣，你应该去看一看]  

# 5. CDNs对移动用户也有效吗？  
[我前几周写过关于这个问题的回答](http://www.webperformancetoday.com/2013/05/09/case-study-cdn-content-delivery-network-mobile-3g/)，但是简短的回答是：**是的，CDNs某种意义上对移动用户有效，但是这也许很难消除边际报酬的消耗。**  
在案例研究中我发现，我们一步步测试网页加载来测量出每一个步骤移动用户的影响。当我们加上CDN以后，我们发现：  
* 文本完成时间降低了10%，在桌面网络情况下降低了20%
* 二次打开的开始渲染时间从7.059秒降低到6.245秒  
所以虽然有性能的提升，但是不够明显。虽然我不想说CDNs对移动端无效，但是当讨论CDN能为移动端提供的优化时，我提供了有利的论点。  

# 6. 用CDN能为网站保证100%的可用性吗？
我遇到的每一个CDN的提供商都保证了接近100%的可用性，即使在巨大的电力中断，硬件故障和网络问题时。这些保障的基础是CDNs有一套自动的机制，当有服务器宕机时，来把将用户的请求重定向到可用的服务器。  

所以，“接近100%的可用性”意味着说明呢？**一个CDN的运行时间应该保证在用户的SLA（服务水平协议）之内**。我也建议如果你对某一个提供商感兴趣，你应该谷歌“[CDN名字] outage availability”然后看看出现了哪种结果。  

# 7. 大多数排名靠前的网站使用CDN吗？  
不。根据最近的调查，排名靠前的电商网站中，欧洲前400的79%，美国前2000的75%不使用被承认的CDN。（被承认的CDN指的是被[WebPagetest](http://www.webpagetest.org/)收录的CDN）。  

费用也许是最有可能的因素了。但是最近的几年，更有竞争力的价格选择开始出现在市场上了。  

# 8. CDNs和前端优化相比，对分发网页有什么区别？ 
为了获得最好的加速效果，**我们的大多客户使用了前端优化（FEO）和内容分发网络（CDN），应用分发控制器（ADC）和内部引擎结合的方案**。  
我们之前已经提到，CDNs针对性能的中间部分（Middle mile区别于Last mile），通过减少资源和用户的物理距离。这样做的结果，让页面的加载更快。FEO在前段提高性能以此来增快浏览器的渲染速度。  

结合FEO和CDN的优势不是新闻了，如果你已经读我（作者）的博客有一阵子。如下表所示（使用了这里的[数据](http://www.webperformancetoday.com/2012/05/14/cdn-feo-front-end-optimization-web-acceleration/)，结合额的解决方案带来了巨大的性能提升，不仅是负载的请求数，开始渲染的时间还是加载的时间。  

![figure](http://ohvmg8dgt.bkt.clouddn.com/CDN+FEO-chart.jpg)  

**CDN/FEO结合的方法可以让页面的加载速度快4倍**  

我们也做了一些实验来在问题#5中，但是对于桌面而不是移动网络。这个逐步加载的视频展示了应用了一系列优化的效果——keep-alives和压缩，CDN和FEO以及跟无优化的对比。  
<iframe width="795" height="597" src="https://www.youtube.com/embed/IPn0T1UacIA" frameborder="0" allowfullscreen></iframe>  
视频中的测试结果：  
* 未优化：16 秒
* keep-alives和压缩： 9 秒
* keep-alives，压缩和CDN: 7 秒
* keep-alives，压缩，CDN和FEO： 3 秒  

使用FEO补充CDN还有些其他优势，如：  
* 重命名文件会导致CDN浪费额外的时间，FEO可以自动解决这个问题，这会节省很多开发的时间。
* 增加超时文件头是优化的最好实践，所以你应该使用。但是增加文件头虽然简单，但是处理版本变化却很复杂，尤其是你在使用一个CDN时。你或者需要额外的人来维护CDN的版本库，或者需要花费巨额的开发时间来整合CDN的API。一个自动的解决方法帮你解决了这些事情，这也节省了大量的开发时间并且减少了可能的错误。  

# 9. CDN可以使用SPDY吗？
谷歌的SPDY是一个网站内容传输的协议，被设计专门用来对付延迟。早先的测试发现SPDY分发的记载时间降低了27%到60%在HTTP上。  

**SPDY没有被广泛地适用在内容分发工业界**。最近的[播客采访](http://www.webperformancetoday.com/podcast/ilya-grigorik-google/)中，谷歌的Ilya Grigorik讨论了为什么广播CDN适用SPDY会是巨大的福利，不只是对SPDY，许多优化的最佳实践方案和最终对终端用户。  

# 10. CDN市场先进的竞争力如何？
现在是成为站长的好机会。几年前，内容分发解决方案是奢侈品，以至只有财大气粗的巨型网站才能够负担。（想想CDN的月账单上十万美元）。**现在越来越多有创新力的公司推出新的技术来提供越来越多有竞争力的选择产品，例如real user monitoring（RUM）。**  

想知道谁是市场的领军者，看一下[知名CDN提供商列表](http://blog.streamingmedia.com/2014/07/cdnvendors.html)。为了分析，Dan Rayburn做了杰出的工作并保持更新来观察CDN领域。如果你对这个领域感兴趣，[他的博客](http://blog.streamingmedia.com/)你不容错过。  

# 11. 我如何为自己的网站/业务选择正确的CDN？
当我准备为这个问题写下回答时，我想起了[Dan写的一篇绝妙的详细博文](http://www.streamingmedia.com/Articles/Editorial/Featured-Articles/Choosing-a-CDN-65114.aspx)。 谢谢你，Dan!  