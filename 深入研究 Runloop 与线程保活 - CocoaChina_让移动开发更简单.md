<!doctype html>
<html>
<head>
    <meta charset="UTF-8">

    <title>深入研究 Runloop 与线程保活 - CocoaChina_让移动开发更简单</title>
    <meta name="keywords" content="Runloop,线程,iOS开发iPhone开发,iOS开发,iPad开发,Mac开发,苹果开发中文站,iPhone开发中文站,CocoaChina首页, Mac OS开发, Cocoa介绍,移动互联网,触控科技,Cocoa,Apple,developer,iOS,iPhone,iPad,iMac,iPod Touch,iPhone5,iPhone4S,iPad3,招聘,iPhone程序员,Objective-c,iPhone应用外包,ios6,ios面试,Cocos2d-x,cocos2d,iTunes,App Store,苹果开发" />
    <meta name="description" content="CocoaChina前身是全球成立最早规模最大的苹果开发中文站，现致力为所有移动开发者提供资讯服务、问答服务、代码下载、工具库及人才招聘服务" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="baidu-tc-cerfication" content="3300a939a1098c27e94457b75a0bb8c1" />
    <script type='text/javascript'>
        function gotoMobilePage() {
            var sUserAgent = navigator.userAgent.toLowerCase();
            var agent_iphone = /iphone/i;
            var agent_android = /android/i;
            //var agent_ipad = /ipad/i;
            var agent_ipod = /ipod/i;
            var agent_wphone = /windows phone/i;
            if(agent_iphone.test(sUserAgent) || agent_android.test(sUserAgent) || agent_ipod.test(sUserAgent) || agent_wphone.test(sUserAgent)){
                var re = /\/\w+\/\d+\/(\d+)\.html/, 
                    result = re.exec(window.location.pathname);
                if ( result )
                {
                    window.location.href = "http://www.cocoachina.com/cms/wap.php?action=article&id=" + result[1];
                } else {
                    window.location.href = "http://m.cocoachina.com/";
                }
            }
        }
        gotoMobilePage(); 
    </script>
    <link rel="stylesheet" href="http://cdn.cocimg.com/assets/css/global.css?v=20150818" media="all">
    <link rel="stylesheet" href="http://cdn.cocimg.com/assets/css/index.css?v=20151015" media="all">
    <link href="http://cdn.cocimg.com/assets/css/module.css?v=20150906" rel="stylesheet">
    <link href="http://cdn.cocimg.com/assets/css/style.css?v=620150818" rel="stylesheet">
    <link href="http://cdn.cocimg.com/assets/css/navheader.css?v=20160301" rel="stylesheet" />
    <script src="http://cdn.cocimg.com/assets/js/jquery.min.js?v=20150818" type="text/javascript"></script>
    <script type="text/javascript" src="http://cdn.cocimg.com/assets/js/navbar.js?v=20151013"></script>
    <script type='text/javascript' src='http://js.adm.cnzz.net/js/abase.js'></script>
    <script type="text/javascript" src="http://s.csbew.com/k.js"></script>
    
</head>
<body>

    <div class="nav-header">
            <div class="nav-header-top">
                <div class="nav-logo nav-fl"><a href="/"><img src="http://cdn.cocimg.com/assets/images/logo.png?v=201510272" /></a></div>
                <div class="nav-list nav-fl">
                    <ul>
                        <li><a href="/news/">资讯</a></li>
                        <li><a href="/bbs/">论坛</a></li>
                        <li><a target="_blank" href="http://code.cocoachina.com/" title="CODE">代码</a></li>
                        <li><a href="http://tools.cocoachina.com" title="工具" target='_blank'>工具</a></li>
                        <li><a target="_blank" href="http://job.cocoachina.com/" title="JOB">招聘</a></li>
                        <li><a target="_blank" href="http://cvp.cocos.com/" title="CVP">CVP</a></li>
                        <li><a target="_blank" href="http://waikuai.cocoachina.com/" title="waikuai">外快</a></li>
                        <li><a target="_blank" href="http://blog.cocoachina.com/" title="blog">博客</a><span class="new-item-flag">new</span></li>
                    </ul>
                </div>
                <div class="nav-search nav-fl">
                    <div class="nav-search-flag" style="display:block"><img src="http://cdn.cocimg.com/assets/img/search-icon.png?v=20150825" /></div>
                    <div class="nav-search-form" style="display:none">
                        <form action="/cms/plus/search.php" name="formsearch" method="post" target="_blank">
                            <input type="hidden" name="kwtype" value="0" />
                            <input type="hidden" name="searchtype" value="titlekeyword" />
                            <input name="keyword" type="text" placeholder="站内搜索" />
                            <input type="button" />
                        </form>
                    </div>
                </div>
                <div class="nav-user nav-fr">
                    <!-- #region 未登录时显示此区域 -->
                    <div class="nav-user-login" style="display:block">
                        <a href="/bbs/login.php">登录</a>|
                        <a href="/bbs/register.php">注册</a>
                    </div>
                    <!-- #endregion -->
                    <!-- #region 已登录时显示此区域 -->
                    <div class="nav-user-name" style="display:none">
                        <div><alert></alert><img src="/assets/img/user.png" />Guest</div>
                        <ul>
                            <li><a href="#"><img src="/assets/img/i-man.png" />我的主页</a></li>
                            <li><alert></alert><a href="#"><img src="/assets/img/i-mail.png" />我的消息</a></li>
                            <li><a href="#"><img src="/assets/img/i-setting.png" />账户设置</a></li>
                            <li><a href="index-offline.html"><img src="/assets/img/i-logout.png" />退出</a></li>
                        </ul>
                    </div>
                    <!-- #endregion -->
                </div>
                <div class="nav-clear"></div>
            </div>
        <div class="nav-header-bottom">
            <ul>
                <li><a id="ios" href="/ios/" title="ios">iOS开发</a></li>
                <li><a id="swift" href="/swift/" title="swift">Swift</a></li>
                <li><a id="app_store" href="/appstore/" title="appstore">App Store研究</a></li>
                <li><a id="design" href="/design/" title="design">产品设计</a></li>
                <li><a id="review" href="/review/" title="review">应用</a></li>
                <li><a id="review" href="/vr/" title="review">VR</a></li>
                <li><a id="game" href="/game/" title="game">游戏开发</a></li>
                <li><a id="apple" href="/apple/" title="apple">苹果相关</a></li>
                <!-- <li><a id="cocos" href="/cocos/" title="cocos">Cocos引擎</a></li>-->
                <!--<li><a id="webapp" href="/webapp/" title="HTML5">HTML5</a></li>-->
                <li><a id="android" href="/android/" title="android">安卓相关</a></li>
                <li><a id="market" href="/market/" title="market">营销推广</a></li>
                <li><a id="industry" href="/industry/" title="industry">业界动态</a></li>
                <li><a id="programmer" href="/programmer/" title="programmer">程序人生</a></li>
            </ul>
        </div>
    </div>

<script language="javascript">
    function IsPC() {
        var userAgentInfo = navigator.userAgent;
        var Agents = new Array("Android", "iPhone", "SymbianOS", "Windows Phone", "iPod");
        var flag = true;
        for (var v = 0; v < Agents.length; v++) {
        if (userAgentInfo.indexOf(Agents[v]) > 0) { flag = false; break; }
        }
        return flag;
    }
    if (!IsPC()){
           window.location.href = "http://www.cocoachina.com/cms/wap.php?action=article&id=17220";
    }

</script>
<!--middle-->
<div class="middle clearfix">
    <div class="m-wrap">
        <div class="m-ad2" style="margin-top:12px;">
            <div class="ad2-l">
                <script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
                <!-- CC首页底部左侧72890 -->
                <ins class="adsbygoogle"
                     style="display:inline-block;width:728px;height:90px"
                     data-ad-client="ca-pub-1180518835303646"
                     data-ad-slot="8012423965"></ins>
                <script>
                (adsbygoogle = window.adsbygoogle || []).push({});
                </script>
            </div>
            <div class="ad2-r">
                <script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
                <!-- CC资讯页25090 -->
                <ins class="adsbygoogle"
                     style="display:inline-block;width:250px;height:90px"
                     data-ad-client="ca-pub-1180518835303646"
                     data-ad-slot="9489157162"></ins>
                <script>
                (adsbygoogle = window.adsbygoogle || []).push({});
                </script>
            </div>
        </div>
        <!--left-->
        <div class="detail-left float-l">
            <p class="crumbs"><a href="/" title="首页">首页</a> &gt;</span><a href="/ios" title="iOS开发"><span id="type_name">iOS开发</span></a></p>
            <div class="detail-main">
                <h2>深入研究 Runloop 与线程保活</h2>
                <div class="p-ico clearfix">
                    <div>
                        <span class="ml0">2016-07-28 07:46</span>
                        <span>编辑：
                        <a href="" id="field_author" target="_blank">cocopeng</a></span>
                        <span>分类：<a href="/ios/" target="_blank">iOS开发</a></span>
                        <span id="source">来源：<a href="http://www.jianshu.com/p/10121d699c32#" target="_blank">bestswifter  简书</a></span>

                        <div class="float-r">
                            <span id="artical_comment_cnt"><i></i></span>
                            <span><i class="view"></i><script src="/cms/plus/count.php?view=yes&aid=17220&mid=8" language="javascript"></script></span>
                    </div>
                    </div>
                </div>
                <div class="p-ico clearfix">
                    <p class="related float-l"><a href="/cms/tags.php?/iOS%E5%BC%80%E5%8F%91/" target="_blank">iOS开发</a><a href="/cms/tags.php?/%E7%BA%BF%E7%A8%8B/" target="_blank">线程</a><a href="/cms/tags.php?/RunLoop/" target="_blank">RunLoop</a></p>
                    <input type="hidden" id="article_id"  value="17220"/>
                </div>
                <div class="adsbox clearfix">
                    <div class="float-l">
                        <div class="float-l"><b>招聘信息：</b></div>
                        <ul class="float-l" id="invitelist" style="margin-top: 0px;">
                            <li><a href='http://job.cocoachina.com/job/show?id=65858' target='_blank'>iOS高级开发工程师</a></li><li><a href='http://job.cocoachina.com/job/show?id=67531' target='_blank'>[成都 | 远程办公] 招聘Android工程师</a></li><li><a href='http://job.cocoachina.com/job/show?id=68061' target='_blank'>资深软件研发工程师</a></li><li><a href='http://job.cocoachina.com/job/show?id=68062' target='_blank'>算法工程师（机器视觉方向）</a></li><li><a href='http://job.cocoachina.com/job/show?id=68063' target='_blank'>基础架构工程师</a></li><li><a href='http://job.cocoachina.com/job/show?id=68064' target='_blank'>后端开发工程师</a></li><li><a href='http://job.cocoachina.com/job/show?id=68065' target='_blank'>Web 前端开发工程师</a></li><li><a href='http://job.cocoachina.com/job/show?id=68067' target='_blank'>应用开发工程师(Android)</a></li><li><a href='http://job.cocoachina.com/job/show?id=68066' target='_blank'>应用开发工程师(iOS)</a></li><li><a href='http://job.cocoachina.com/job/show?id=68068' target='_blank'>应用开发工程师(机器人应用)</a></li><li><a href='http://job.cocoachina.com/job/show?id=68069' target='_blank'>高级产品经理</a></li>
                            <!--<script src="http://www.cocoachina.com/bbs/new.php?action=article&fidin=26&digest=0&postdate=0&author=0&fname=0&hits=0&replies=0&pre=1&num=3&length=120&order=2" charset="gbk"></script>-->
                        </ul>
                    </div>
                    <div class="float-r">
                        <a href="javascript:void(0)" class="prev"></a>
                        <a href="javascript:void(0)" class="next"></a>
                    </div>
                </div>
                <div id="detailbody" class="field_body">
                    <p style="text-align:center"><img src="http://cc.cocimg.com/api/uploads/20160727/1469613145939306.jpg" title="1469613145939306.jpg" alt="1441627490-runloop3600.jpg"/></p><p>授权转载，作者：<a href="http://weibo.com/u/5678670890" target="_blank">bestswifter</a></p><p>在讨论 runloop 相关的文章，以及分析 AFNetworking(2.x) 源码的文章中，我们经常会看到关于利用 runloop 进行线程保活的分析，但如果不求甚解的话，极有可能因此学会了一个错误的用法，本文就来分析一下其中常见的误区。</p><p>我提供了一个 Demo，可以在我的 Github 上下载并运行一遍，文章中只提供了部分代码。</p><p>Demo地址：<a href="https://github.com/bestswifter/MySampleCode/tree/master/RunloopAndThread" target="_blank" textvalue="https://github.com/bestswifter/MySampleCode/tree/master/RunloopAndThread">https://github.com/bestswifter/MySampleCode/tree/master/RunloopAndThread</a> </p><p><strong><span style="line-height: 1.8;">AFN 中的实现</span></strong><br/></p><p>首先我们知道在<a href="https://github.com/AFNetworking/AFNetworking/tree/2.x" target="_blank" textvalue="旧版本的AFN">旧版本的AFN</a>中使用了 NSURLConnection 来发起并处理网络连接。AFN 的做法是把网络请求的发起和解析都放在同一个子线程中进行，但由于子线程默认不开启 runloop，它会向一个 C语言程序那样在运行完所有代码后退出线程。而网络请求是异步的，这会导致获取到请求数据时，线程已经退出，代理方法没有机会执行。因此，AFN 的做法是使用一个 runloop 来保证线程不死，也就是下面这段被讲烂了的代码:</p><pre class="brush:js;toolbar:false">+&nbsp;(void)networkRequestThreadEntryPoint:(id)__unused&nbsp;object&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;@autoreleasepool&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[[NSThread&nbsp;currentThread]&nbsp;setName:@"AFNetworking"];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NSRunLoop&nbsp;*runLoop&nbsp;=&nbsp;[NSRunLoop&nbsp;currentRunLoop];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[runLoop&nbsp;addPort:[NSMachPort&nbsp;port]&nbsp;forMode:NSDefaultRunLoopMode];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[runLoop&nbsp;run];
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre><p>当然，单独看这一个方法意义不大，我们稍微结合一下上下文，看看这个方法在哪里被调用:</p><pre class="brush:js;toolbar:false">+&nbsp;(NSThread&nbsp;*)networkRequestThread&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;static&nbsp;NSThread&nbsp;*_networkRequestThread&nbsp;=&nbsp;nil;
&nbsp;&nbsp;&nbsp;&nbsp;static&nbsp;dispatch_once_t&nbsp;oncePredicate;
&nbsp;&nbsp;&nbsp;&nbsp;dispatch_once(&oncePredicate,&nbsp;^{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_networkRequestThread&nbsp;=&nbsp;[[NSThread&nbsp;alloc]&nbsp;initWithTarget:self&nbsp;selector:@selector(networkRequestThreadEntryPoint:)&nbsp;object:nil];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[_networkRequestThread&nbsp;start];
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;_networkRequestThread;
}</pre><p>似乎这种写法提供了一种思路:“如果需要在子线程中异步执行操作，可以利用 runloop 进行线程保活”。但准确的来说，AFN 的这种写法并不能实现我们的需求，它只是在 AFN 这个特殊场景下可以工作。</p><p>不信你可以尝试阅读一下第二段代码，看看它和平时使用 NSThread 时有什么区别，如果没看出来也无妨，先记住这段代码，我们稍后分析。</p><p><strong>NSThread 与内存泄漏</strong></p><p>这种写法的第一个问题就是存在内存泄漏。我们构造以下用例，其实就是把 AFN 的线程创建放在一个循环里:</p><pre class="brush:js;toolbar:false">-&nbsp;(void)memoryTest&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(int&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;<&nbsp;100000;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NSThread&nbsp;*thread&nbsp;=&nbsp;[[NSThread&nbsp;alloc]&nbsp;initWithTarget:self&nbsp;selector:@selector(run)&nbsp;object:nil];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[thread&nbsp;start];
&nbsp;&nbsp;&nbsp;&nbsp;}
}
-&nbsp;(void)run&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;@autoreleasepool&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NSLog(@"current&nbsp;thread&nbsp;=&nbsp;%@",&nbsp;[NSThread&nbsp;currentThread]);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NSRunLoop&nbsp;*runLoop&nbsp;=&nbsp;[NSRunLoop&nbsp;currentRunLoop];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(!self.emptyPort)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;self.emptyPort&nbsp;=&nbsp;[NSMachPort&nbsp;port];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[runLoop&nbsp;addPort:self.emptyPort&nbsp;forMode:NSDefaultRunLoopMode];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[runLoop&nbsp;run];
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre><p>奇怪的事情出现了，尽管是在 ARC 环境下，内存依然不停的上涨。如果我们把 run 方法中和 runloop 相关的代码删除则不会出现上述问题，显然，开启 runloop 导致了内存泄漏，也就是 thread 对象无法释放。</p><p>这里的 emptyPort 用来维持 runloop 的运行，根据官方文档的描述，如果 runloop 中没有任何 modeItem，就不会启动，而是立刻退出。之所以选择作为属性而不是临时变量，是因为我发现每次调用 [NSMachPort port] 方法都会占用内存，原因暂时不清楚。</p><p>我们可以尝试手动结束 runloop 并关闭线程:</p><pre class="brush:js;toolbar:false">-&nbsp;(void)memoryTest&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(int&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;<&nbsp;100000;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NSThread&nbsp;*thread&nbsp;=&nbsp;[[NSThread&nbsp;alloc]&nbsp;initWithTarget:self&nbsp;selector:@selector(run)&nbsp;object:nil];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[thread&nbsp;start];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[self&nbsp;performSelector:@selector(stopThread)&nbsp;onThread:thread&nbsp;withObject:nil&nbsp;waitUntilDone:YES];
&nbsp;&nbsp;&nbsp;&nbsp;}
}
-&nbsp;(void)stopThread&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;CFRunLoopStop(CFRunLoopGetCurrent());
&nbsp;&nbsp;&nbsp;&nbsp;NSThread&nbsp;*thread&nbsp;=&nbsp;[NSThread&nbsp;currentThread];
&nbsp;&nbsp;&nbsp;&nbsp;[thread&nbsp;cancel];
}</pre><p>很遗憾，这依然没有任何效果。而且不难猜测是我们没有能正确的结束 runloop 的运行。</p><p><strong>Runloop 的启动与退出</strong></p><p>考验英文水平的时候到了，首先来看一段<a href="https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW1" target="_blank">官方文档</a>对于如何启动 runloop 的介绍，它的启动方式一共有三种:</p><ul class=" list-paddingleft-2" ><li><p>Unconditionally</p></li><li><p>With a set time limit</p></li><li><p>In a particular mode</p></li></ul><p>这三种进入方式分别对应了三种方法，其中第一种就是我们目前使用的:</p><ul class=" list-paddingleft-2" ><li><p>run</p></li><li><p>runUntilDate</p></li><li><p>runMode:beforeDate:</p></li></ul><p>接下来分别是对三种方式的介绍，文字比较啰嗦，这里我简单总结一下，有兴趣的读者可以直接看原文。</p><ul class=" list-paddingleft-2" ><li><p>无条件进入是最简单的做法，但也最不推荐。这会使线程进入死循环，从而不利于控制 runloop，结束 runloop 的唯一方式是 kill 它。</p></li><li><p>如果我们设置了超时时间，那么 runloop 会在处理完事件或超时后结束，此时我们可以选择重新开启 runloop。这种方式要优于前一种</p></li><li><p>这是相对来说最优秀的方式，相比于第二种启动方式，我们可以指定 runloop 以哪种模式运行。</p></li></ul><p>查看 run 方法的文档还可以知道，它的本质就是无限调用 runMode:beforeDate: 方法，同样地，runUntilDate: 也会重复调用 runMode:beforeDate:，区别在于它超时后就不会再调用。</p><p><strong>总结来说，runMode:beforeDate: 表示的是 runloop 的单次调用，另外两者则是循环调用。</strong></p><p>相比于 runloop 的启动，它的退出就比较简单了，只有两种方法:</p><ul class=" list-paddingleft-2" ><li><p>设置超时时间</p></li><li><p>手动结束</p></li></ul><p>如果你使用方法二或三来启动 runloop，那么在启动的时候就可以设置超时时间。然而考虑到目标是:“利用 runloop 进行线程保活”，所以我们希望对线程和它的 runloop 有最精确的控制，比如在完成任务后立刻结束，而不是依赖于超时机制。</p><p>好在根据文档的描述，我们还可以使用 CFRunLoopStop() 方法来手动结束一个 runloop。注意文档中在介绍利用 CFRunLoopStop() 手动退出时有下面这句话:</p><p>The difference is that you can use this technique on run loops you started unconditionally.</p><p>这里的解释非常容易产生误会，如果在阅读时没有注意到 exit 和 terminate 的微小差异就很容易掉进坑里，因为在 run 方法的文档中还有这句话:</p><p>If you want the run loop to terminate, you shouldn&#39;t use this method</p><p>总的来说，如果你还想从 runloop 里面退出来，就不能用 run 方法。根据实践结果和文档，另外两种启动方法也无法手动退出。</p><p><strong>正确的做法</strong></p><p>难道子线程中开启了 runloop 就无法结束并释放了么？这显然是一个不合理的结论，经过一番查找，终于在<a href="http://www.cocoabuilder.com/archive/cocoa/305204-runloop-not-being-stopped-by-cfrunloopstop.html" target="_blank">这篇文章</a>里找到了答案，它给出了使用 CFRunLoopStop() 无效的原因:</p><p>CFRunLoopStop() 方法只会结束当前的 runMode:beforeDate: 调用，而不会结束后续的调用。</p><p>这也就是为什么 Runloop 的文档中说 CFRunLoopStop() 可以 exit(退出) 一个 runloop，而在 run 等方法的文档中又说这样会导致 runloop 无法 terminate(终结)。</p><p>文章中给出的方案是使用 CFRunLoopRun() 启动 runloop，这样就可以通过 CFRunLoopStop() 方法结束。而文档则推荐了另一种方法:</p><pre class="brush:js;toolbar:false">BOOL&nbsp;shouldKeepRunning&nbsp;=&nbsp;YES;&nbsp;//&nbsp;global
NSRunLoop&nbsp;*theRL&nbsp;=&nbsp;[NSRunLoop&nbsp;currentRunLoop];
while&nbsp;(shouldKeepRunning&nbsp;&&&nbsp;[theRL&nbsp;runMode:NSDefaultRunLoopMode&nbsp;beforeDate:[NSDate&nbsp;distantFuture]]);</pre><p>我尝试了文档提供的方法，确实不会导致内存泄漏，但不方便验证 runloop 是否真的开启，然后又被终止。所以我实际采用的是第一种方案:</p><pre class="brush:js;toolbar:false">-&nbsp;(void)memoryTest&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(int&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;<&nbsp;100000;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NSThread&nbsp;*thread&nbsp;=&nbsp;[[NSThread&nbsp;alloc]&nbsp;initWithTarget:self&nbsp;selector:@selector(run)&nbsp;object:nil];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[thread&nbsp;start];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[self&nbsp;performSelector:@selector(stopThread)&nbsp;onThread:thread&nbsp;withObject:nil&nbsp;waitUntilDone:YES];
&nbsp;&nbsp;&nbsp;&nbsp;}
}
-&nbsp;(void)stopThread&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;CFRunLoopStop(CFRunLoopGetCurrent());
&nbsp;&nbsp;&nbsp;&nbsp;NSThread&nbsp;*thread&nbsp;=&nbsp;[NSThread&nbsp;currentThread];
&nbsp;&nbsp;&nbsp;&nbsp;[thread&nbsp;cancel];
}
-&nbsp;(void)run&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;@autoreleasepool&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NSLog(@"current&nbsp;thread&nbsp;=&nbsp;%@",&nbsp;[NSThread&nbsp;currentThread]);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NSRunLoop&nbsp;*runLoop&nbsp;=&nbsp;[NSRunLoop&nbsp;currentRunLoop];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(!self.emptyPort)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;self.emptyPort&nbsp;=&nbsp;[NSMachPort&nbsp;port];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[runLoop&nbsp;addPort:self.emptyPort&nbsp;forMode:NSDefaultRunLoopMode];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[runLoop&nbsp;runMode:NSRunLoopCommonModes&nbsp;beforeDate:[NSDate&nbsp;distantFuture]];
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre><p><strong>验证</strong></p><p>采用上述方案后，确实可以观察到不会再出现内存泄漏问题，但这并不是终点。因为我们还需要验证 runloop 确实在启动后被关闭。</p><p>为了证明 runloop 确实启动，我设计了如下方法:</p><pre class="brush:js;toolbar:false">-&nbsp;(void)printSomething&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;NSLog(@"current&nbsp;thread&nbsp;=&nbsp;%@",&nbsp;[NSThread&nbsp;currentThread]);
&nbsp;&nbsp;&nbsp;&nbsp;[self&nbsp;performSelector:@selector(printSomething)&nbsp;withObject:nil&nbsp;afterDelay:1];
}</pre><p>我们知道 performSelector:withObject:afterDelay 依赖于线程的 runloop，因为它本质上是由一个定时器负责定期加入到 runloop 中执行。所以如果这个方法可以成功执行，说明当前线程的 runloop 已经开启，否则则说明没有启动。</p><p>为了证明 runloop 可以被终止，我创建了一个按钮，在点击按钮时执行以下方法:</p><pre class="brush:js;toolbar:false">-&nbsp;(void)stopButtonDidClicked:(id)sender&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;[self&nbsp;performSelector:@selector(stopRunloop)&nbsp;onThread:self.thread&nbsp;withObject:nil&nbsp;waitUntilDone:YES];
}
-&nbsp;(void)stopRunloop&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;CFRunLoopStop(CFRunLoopGetCurrent());
}</pre><p>成功的观察到点击按钮后，控制台不再有日志输出，因此证明 runloop 确实已经停止。</p><p><strong>总结</strong></p><p>啰嗦了这么多，其实是为了研究如何利用 runloop 实现线程保活。要注意的地方主要有以下点:</p><ul class=" list-paddingleft-2" ><li><p>了解 runloop 实现线程保活的原理，注意添加的那个空 port</p></li><li><p>了解 runloop 导致的线程对象内存泄漏问题</p></li><li><p>了解 runloop 的几种启动方式以及彼此之间的关联</p></li><li><p>了解 runloop 的释放方式和原理</p></li></ul><p>由于相关资料的匮乏以及个人水平有限，虽然竭力研究但仍不保证绝对的正确性，欢迎交流指正。</p><p>最后，文章开头对 AFN 的分析留作一个简单的思考题，为什么 AFN 中的用法不会有问题？</p><p><strong>参考资料</strong></p><p><a href="https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html" target="_blank">Run Loops 官方文档</a></p><p><a href="http://www.cocoabuilder.com/archive/cocoa/305204-runloop-not-being-stopped-by-cfrunloopstop.html" target="_blank">Runloop not being stopped by CFRunLoopStop?</a></p><p><a href="http://blog.ibireme.com/2015/05/18/runloop/" target="_blank">深入理解 RunLoop</a></p>
                </div>
                <div id="page" style="padding:0"></div>

                <div class="wx_article">
                    <div class="article_weixin">
                        <img src="http://cdn.cocimg.com/cms/templets/images/article_weixin.jpg" width="131" height="131" alt="搜索CocoaChina微信公众号：CocoaChina"/>
                        <div class="article_scan">微信扫一扫</div>
                        <div class="article_txt" ><span style="color:#999999">订阅每日移动开发及APP推广热点资讯<br/>公众号：</span>CocoaChina</div>
                    </div>
                </div>

               <div class="clearfix tgbox">
                    <a href="http://www.cocoachina.com/deliver.php" class="tg float-l" target="_blank">我要投稿</a>&nbsp;&nbsp;
                    <a style="margin-left:10px"  href="javascript:void(0)" onClick="add_favors(17220,'深入研究 Runloop 与线程保活')" class="tg float-l" >收藏文章</a>
                    <div class="share float-r">分享到：
                        <div class="bdsharebuttonbox" id="sharediv"><a href="#" class="bds_more" data-cmd="more"></a> <a href="#" class="bds_qzone" data-cmd="qzone" title="分享到QQ空间"></a><a href="#" class="bds_tsina" data-cmd="tsina" title="分享到新浪微博"></a> <a href="#" class="bds_weixin" data-cmd="weixin" title="分享到微信"></a></div>
                    </div>
                </div>


                <a class="zanbox" href="javascript:void(0)" id="zanbox" onClick="postDigg('good',17220)" title="哎呦，不错噢，赞一个" >
                    <strong id="newdigg">
                     <script src="/cms/plus/count.php?view=yes&digg_id=17220&mid=8" language="javascript"></script>
                    </strong>
                </a>
            </div>

            <div class="article-prev">上一篇：<a href='/ios/20160727/17215.html'>iOS 最详细的解析（数组与指针）笔试题</a> </div>
            <div class="article-next"></div>

            <!--相关资讯-->
            <div class="part-wrap clearfix">
                <h3>相关资讯</h3>
                <ul class="info-list clearfix">
                    <li>
                        <a href="/ios/20160727/17215.html" target="_blank" title="这个笔试题想必很多小伙伴都很面熟把，差不多10个人有7个人不会做这道笔试题，或许有知道答案的，但是仅仅知道答案，心里还是一头雾水。如果你真的不会那就请认真看完本文章学习一下吧！">iOS 最详细的解析（数组与指针）笔试题</a>
                     </li>
<li>
                        <a href="/ios/20160727/17203.html" target="_blank" title="很多应用都需要用到检测屏幕帧率的功能，用以统计监测应用的流畅度，今天就来介绍一个很有用的第三方开源库——ALFPSStatus。本文从原理以及过程中遇到的问题逐一剖析。">你不知道的第三方开源库——ALFPSStatus</a>
                     </li>
<li>
                        <a href="/ios/20160726/17194.html" target="_blank" title="在上一篇文章《使用AVPlayer播放网络音乐》介绍了AVPlayer的基本使用，下面介绍如何通过AVAssetResourceLoader实现AVPlayer的缓存。">iOS音频篇：AVPlayer的缓存实现</a>
                     </li>
<li>
                        <a href="/ios/20160726/17189.html" target="_blank" title="本文主要涉及到两个概念： Emulator 和 Simulator。通常我们在工作中可能统统习惯称为“模拟器”，但实际上二者有所不同。为了分清概念，本文将 Emulator 译作 “仿真器”， Simulator 译作 “模拟器”。">移动开发中的仿真器与模拟器</a>
                     </li>
<li>
                        <a href="/ios/20160726/17174.html" target="_blank" title="何为视觉差,当初找效果的时候，也不知道如何搜索，后来知道了视差Parallax这个词，我这里写的效果是，在页面滚动的时候，每个cell中的图片也会产生位置上的变化，给人一种视觉插得感觉，废话不多">iOS视觉差Parallax效果简析</a>
                     </li>
<li>
                        <a href="/ios/20160725/17172.html" target="_blank" title="之前看见过很多动画, 都非常炫酷, 所以想弄一些比较简单的动画, 以后再项目中可能会用到, 以后还会持续更新类似的动画效果！">iOS TableView给力动画的简单实现(一)</a>
                     </li>
<li>
                        <a href="/ios/20160725/17163.html" target="_blank" title="在iOS 9.0之前，Apple Spotlight仅能够检索iOS自身应用的内容，比如邮件、备忘录、提醒、短信。第三方应用不支持被检索，比如美团、大众点评、今日头条等等。在iOS9.0之后，iOS苹果推出Search API，使得第">Core Spotlight和深度链接结合使用(上)</a>
                     </li>
<li>
                        <a href="/ios/20160722/17144.html" target="_blank" title="移动应用实现深度链接需要考虑非常多复杂的情况，比如支持各种手机机型、移动操作系统、浏览器、系统版本等等，还要考虑到深度链接统计分析的诸多问题。">Deep Linking技术，你真的知道吗</a>
                     </li>
<li>
                        <a href="/ios/20160721/17134.html" target="_blank" title="MapKit框架使用须知：MapKit框架中所有数据类型的前缀都是MK；MapKit有一个比较重要的UI控件 ：MKMapView，专门用于地图显示。">iOS开发之MapKit框架的使用</a>
                     </li>
<li>
                        <a href="/ios/20160721/17133.html" target="_blank" title="技术实现层面来讲，直播APP技术比较成熟，设备也都支持硬编码。iOS还提供现成的Video ToolBox框架，可以对摄像头和流媒体数据结构进行处理，但Video ToolBox框架只兼容8.0以上版本，8.0以下就需要用x264的">做一款仿映客的直播App？看这篇就够了</a>
                     </li>

                </ul>
            </div>
            <div class="clearfix" style="margin-top:20px;" >
                <script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
                <!-- CC评论框 -->
                <ins class="adsbygoogle"
                     style="display:inline-block;width:750px;height:90px"
                     data-ad-client="ca-pub-1180518835303646"
                     data-ad-slot="8242010366"></ins>
                <script>
                    (adsbygoogle = window.adsbygoogle || []).push({});
                </script>
            </div>
            <!-- 评论模块start -->
            
            <div class="clearfix" id="commnet_list_holder" style="margin-bottom:50px;">
                <div class="glab-comment-form">
                    <div class="dcbt">我来说两句</div>
                    <div class="user-head">
                        <img id="user_face_url" src="http://cdn.cocimg.com/bbs/images/face/none.gif" height="43" width="43" alt="">
                    </div>
                    <div class="form-region">
                        <textarea name="comment" id="main_comment_content" cols="30" rows="10" placeholder="你怎么看？快来评论一下吧！"></textarea>
                        <a class="submitform" id="main_comment_bt">发表评论</a>
                        <div class="unlogin-mask">
                            <div>您还没有登录！请<a href="/bbs/login.php">登录</a>或<a href="/bbs/register.php">注册</a></div>
                        </div>
                    </div>
                </div>
                <div id="glab-comment-callback-msg" class="glab-comment-info"></div>
                <div class="glab-comment-title">所有评论（<span>0</span>）</div>
                <div class="glab-commentlist" id="commentlist">
                    <a id="refresh_comment_btn" href="javascript:;" onclick="$('#morecomment').trigger('click')">刷新评论</a>
                </div>
                <div class="glab-comment-more" id="morecomment" style="display:none">
                    更多评论
                </div>
            </div>
            
            <!-- 评论模块end -->


        </div>
        <!--./left-->
        <!--侧边栏-->
        <div class="clearfix newslist">
            <div class="rightSide detail-right float-r"  >
                <!--两周热门资讯推荐-->
                <div>
                    <div class="part-wrap clearfix">
                        <h3>热门资讯</h3>
                        <ul class="related-hot">
                            <li class="clearfix"><a href="http://www.cocoachina.com/ios/20160714/17038.html" class="float-l pic" target="_blank" title="Xcode 8 的 6 大新功能一览"><img src="http://cc.cocimg.com/api/uploads/160713/b67bcae72935c94059a748239e52cba9.jpg"  style="max-height:60px" alt="Xcode 8 的 6 大新功能一览"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/ios/20160714/17038.html" target="_blank" title="Xcode 8 的 6 大新功能一览">Xcode 8 的 6 大新功能一览</a></p><p class="hits">点击量<span>8509</span></p></div></li><li class="clearfix"><a href="http://www.cocoachina.com/ios/20160714/17045.html" class="float-l pic" target="_blank" title="一个收集了502款开源iOS应用的开源项目"><img src="http://cc.cocimg.com/api/uploads/160714/ed7dfecf7e5c81f6528548432c5799c6.jpg"  style="max-height:60px" alt="一个收集了502款开源iOS应用的开源项目"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/ios/20160714/17045.html" target="_blank" title="一个收集了502款开源iOS应用的开源项目">一个收集了502款开源iOS应用的开源项目</a></p><p class="hits">点击量<span>8212</span></p></div></li><li class="clearfix"><a href="http://www.cocoachina.com/programmer/20160718/17061.html" class="float-l pic" target="_blank" title="开发者MAC电脑里的十八般兵器"><img src="http://cc.cocimg.com/api/uploads/160717/6d8d88d8958dafba46cc940edba5e123.jpg"  style="max-height:60px" alt="开发者MAC电脑里的十八般兵器"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/programmer/20160718/17061.html" target="_blank" title="开发者MAC电脑里的十八般兵器">开发者MAC电脑里的十八般兵器</a></p><p class="hits">点击量<span>7695</span></p></div></li><li class="clearfix"><a href="http://www.cocoachina.com/ios/20160721/17133.html" class="float-l pic" target="_blank" title="做一款仿映客的直播App？看这篇就够了"><img src="http://cc.cocimg.com/api/uploads/160721/b2e6a9731c4f7f359254d6413e079cbb.jpg"  style="max-height:60px" alt="做一款仿映客的直播App？看这篇就够了"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/ios/20160721/17133.html" target="_blank" title="做一款仿映客的直播App？看这篇就够了">做一款仿映客的直播App？看这篇就够了</a></p><p class="hits">点击量<span>7435</span></p></div></li><li class="clearfix"><a href="http://www.cocoachina.com/ios/20160719/17104.html" class="float-l pic" target="_blank" title="iOS开发的10个奇袭"><img src="http://cc.cocimg.com/api/uploads/160719/bdde29a29547a43db3ad4aa09cb20dd8.jpg"  style="max-height:60px" alt="iOS开发的10个奇袭"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/ios/20160719/17104.html" target="_blank" title="iOS开发的10个奇袭">iOS开发的10个奇袭</a></p><p class="hits">点击量<span>6678</span></p></div></li><li class="clearfix"><a href="http://www.cocoachina.com/programmer/20160715/17048.html" class="float-l pic" target="_blank" title="15 张令人喷饭的 IT 趣图（第2季）"><img src="http://cc.cocimg.com/api/uploads/160714/4f2e5cf8a7266600f6b1c229ac05e021.jpg"  style="max-height:60px" alt="15 张令人喷饭的 IT 趣图（第2季）"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/programmer/20160715/17048.html" target="_blank" title="15 张令人喷饭的 IT 趣图（第2季）">15 张令人喷饭的 IT 趣图（第2季）</a></p><p class="hits">点击量<span>6472</span></p></div></li><li class="clearfix"><a href="http://www.cocoachina.com/swift/20160713/17028.html" class="float-l pic" target="_blank" title="Swift 3中的新特性"><img src="http://cc.cocimg.com/api/uploads/160712/47e1c1c4c29c65bce3207b121ecfacdb.jpg"  style="max-height:60px" alt="Swift 3中的新特性"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/swift/20160713/17028.html" target="_blank" title="Swift 3中的新特性">Swift 3中的新特性</a></p><p class="hits">点击量<span>5611</span></p></div></li><li class="clearfix"><a href="http://www.cocoachina.com/ios/20160719/17078.html" class="float-l pic" target="_blank" title="欲先攻其事必先利其器 (第三方资源篇)"><img src="http://cc.cocimg.com/api/uploads/160717/8f3a395ed4c5c01f933dedbc8631321f.jpg"  style="max-height:60px" alt="欲先攻其事必先利其器 (第三方资源篇)"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/ios/20160719/17078.html" target="_blank" title="欲先攻其事必先利其器 (第三方资源篇)">欲先攻其事必先利其器 (第三方资源篇)</a></p><p class="hits">点击量<span>5598</span></p></div></li><li class="clearfix"><a href="http://www.cocoachina.com/ios/20160715/17022.html" class="float-l pic" target="_blank" title="iOS开发中WiFi相关功能总结"><img src="http://cc.cocimg.com/api/uploads/160714/5374e9ee44ad6eb8ad713eeb09a82ad5.jpg"  style="max-height:60px" alt="iOS开发中WiFi相关功能总结"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/ios/20160715/17022.html" target="_blank" title="iOS开发中WiFi相关功能总结">iOS开发中WiFi相关功能总结</a></p><p class="hits">点击量<span>5522</span></p></div></li><li class="clearfix noborder-b"><a href="http://www.cocoachina.com/ios/20160718/17094.html" class="float-l pic" target="_blank" title="iOS开发之Masonry框架源码深度解析"><img src="http://cc.cocimg.com/api/uploads/160718/dce7867b21c0fb408208901eb4b8a0a6.jpg"  style="max-height:60px" alt="iOS开发之Masonry框架源码深度解析"/></a><div style="margin-left:86px"><p class="related-intro"><a href="http://www.cocoachina.com/ios/20160718/17094.html" target="_blank" title="iOS开发之Masonry框架源码深度解析">iOS开发之Masonry框架源码深度解析</a></p><p class="hits">点击量<span>4687</span></p></div></li>
                        </ul>
                    </div>
               </div>
               <!--./两周热门资讯推荐-->
               <!--综合评论-->
                <div class="part-wrap clearfix" style="margin-top: 15px;">
                    <h3>综合评论</h3>
                    <ul class="related-comment clearfix" id="all_comments_holder">
                        <li><span class="hide_comment">mark</span><br/><p><span>songleicoco</span><span>评论了</span><a href="http://www.cocoachina.com/ios/20160726/17194.html" title="iOS音频篇：AVPlayer的缓存实现" target="_blank" ><span>iOS音频篇：AVPlayer的缓存实现</span></a></p></li><li><span class="hide_comment">6666</span><br/><p><span>13313325011</span><span>评论了</span><a href="http://www.cocoachina.com/ios/20160726/17192.html" title="iOS 仿YY直播心形动画 & 烟花动画" target="_blank" ><span>iOS 仿YY直播心形动画 & 烟花动画</span></a></p></li><li><span class="hide_comment">厉害了</span><br/><p><span>Dcook</span><span>评论了</span><a href="http://www.cocoachina.com/ios/20160727/17211.html" title="iOS 雪花动画与跑马灯" target="_blank" ><span>iOS 雪花动画与跑马灯</span></a></p></li><li><span class="hide_comment">这个逼装的漂亮，多了作别林在里面</span><br/><p><span>kiss_zhan</span><span>评论了</span><a href="http://www.cocoachina.com/ios/20160727/17199.html" title="谈 UIView Animation 编程艺术" target="_blank" ><span>谈 UIView Animation 编程艺术</span></a></p></li><li><span class="hide_comment">好难！</span><br/><p><span>七月天熊</span><span>评论了</span><a href="http://www.cocoachina.com/apple/20160325/15754.html" title="博客推荐：ApplePay应用内支付线上接入教程" target="_blank" ><span>博客推荐：ApplePay应用内支付线上接入教程</span></a></p></li><li><span class="hide_comment">装逼遭雷劈</span><br/><p><span>fengzhenxiang</span><span>评论了</span><a href="http://www.cocoachina.com/ios/20160612/16653.html" title="4道过滤菜鸟的iOS面试题" target="_blank" ><span>4道过滤菜鸟的iOS面试题</span></a></p></li><li><span class="hide_comment">Xib模块化开发,非常实用&nbsp;&nbsp;&nbsp;http://00red.com/blog/2016/07/27/tips-swift-xib-modular-design/</span><br/><p><span>g1jun</span><span>评论了</span><a href="http://www.cocoachina.com/ios/20111023/3408.html" title="自定义UIViewController与xib文件..." target="_blank" ><span>自定义UIViewController与xib文件...</span></a></p></li><li><span class="hide_comment">Xib模块化开发,非常实用&nbsp;&nbsp;&nbsp;http://00red.com/blog/2016/07/27/tips-swift-xib-modular-design/</span><br/><p><span>g1jun</span><span>评论了</span><a href="http://www.cocoachina.com/ios/20140102/7640.html" title=" 代码手写UI，xib和StoryBoard间的博..." target="_blank" ><span> 代码手写UI，xib和StoryBoard间的博...</span></a></p></li><li><span class="hide_comment">Xib模块化开发,非常实用&nbsp;&nbsp;&nbsp;http://00red.com/blog/2016/07/27/tips-swift-xib-modular-design/</span><br/><p><span>g1jun</span><span>评论了</span><a href="http://www.cocoachina.com/ios/20140530/8620.html" title="结队开发之多storyboard" target="_blank" ><span>结队开发之多storyboard</span></a></p></li><li class="noborder-b"><span class="hide_comment">这有一个特别实用的技术，Xib模块化开发http://00red.com/blog/2016/07/27/tips-swift-xib-modular-design/</span><br/><p><span>g1jun</span><span>评论了</span><a href="http://www.cocoachina.com/design/20160713/17032.html" title="一稿适配所有iOS设备——AutoLayout入门" target="_blank" ><span>一稿适配所有iOS设备——AutoLayout入门</span></a></p></li>
                    </ul>
                </div>
                <!--./综合评论-->
                <!--相关帖子-->
                <div class="part-wrap clearfix" style="margin:15px 0 15px 0;">
                    <h3>相关帖子</h3>
                    <ul class="related-comment clearfix">
                        <script charset="gb2312" src="http://www.cocoachina.com/bbs/new.php?action=article&fidin=21_18_2_8_19_41_59_48_56_14_36_5&digest=0&postdate=0&author=0&fname=0&hits=0&replies=0&pre=0&num=10&length=100&order=2&like=ios#iOS开发"></script>
                    </ul>
                </div>
                <!--./相关帖子-->
                <iframe width="230" height="550" class="share_self" frameborder="0" scrolling="no" src="http://widget.weibo.com/weiboshow/index.php?language=&width=230&height=550&fansRow=2&ptype=1&speed=0&skin=5&isTitle=1&noborder=1&isWeibo=1&isFans=0&uid=1659808677&verifier=6043be56&dpc=1"></iframe>
                <script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
                <!-- CC列表页面230x230 -->
                <ins class="adsbygoogle"
                     style="display:inline-block;width:230px;height:230px"
                     data-ad-client="ca-pub-1180518835303646"
                     data-ad-slot="6765277166"></ins>
                <script>
                (adsbygoogle = window.adsbygoogle || []).push({});
                </script>
            </div>
        </div>
        <!--./侧边栏-->
    </div>

    <div id="coco-store" style="display:none">
        <div class="storebox">
            <h5>提示</h5>
            <h3>已经收藏该篇文章</h3>
            <p class="p1"><a href="/bbs/u.php?action=favor&tag=1" target="_blank">去我的收藏夹</a></p>
            <p class="p2"><a href="/bbs/login.php">请登录</a></p>
        </div>
    </div> <!--收藏提示-->
</div>

<script type="text/javascript">

function postDigg(ftype,aid)
{
    var taget_obj = document.getElementById('newdigg');
    var saveid = GetCookie('diggid');
    if(saveid != null)
    {
        var saveids = saveid.split(',');
        var hasid = false;
        saveid = '';
        j = 1;
        for(i=saveids.length-1;i>=0;i--)
        {
            if(saveids[i]==aid && hasid) continue;
            else {
                if(saveids[i]==aid && !hasid) hasid = true;
                saveid += (saveid=='' ? saveids[i] : ','+saveids[i]);
                j++;
                if(j==20 && hasid) break;
                if(j==19 && !hasid) break;
            }
        }
        if(hasid) { alert("您已经顶过该帖，请不要重复顶帖 ！"); return; }
        else saveid += ','+aid;
        SetCookie('diggid',saveid,1);
    }
    else
    {
        SetCookie('diggid',aid,1);
    }
    myajax = new DedeAjax(taget_obj,false,false,'','','');
    var url = "/cms/plus/digg_ajax.php?action="+ftype+"&id="+aid;
    myajax.SendGet2(url);
    DedeXHTTP = null;
    $('#zanbox').addClass('zanbox-selected');
}
function getDigg(aid)
{
    var taget_obj = document.getElementById('newdigg');
    myajax = new DedeAjax(taget_obj,false,false,'','','');
    myajax.SendGet2("/cms/plus/digg_ajax.php?id="+aid);
    DedeXHTTP = null;
}


//显示对应的导航：
$(function(){
    var article_id = $('#article_id').val();
    var saveid = GetCookie('diggid');

    if(saveid != null)
    {
        var saveids = saveid.split(',');
        var hasid = false;
        saveid = '';
        j = 1;
        for(i=saveids.length-1;i>=0;i--)
        {
            if(saveids[i]==article_id && hasid) continue;
            else {
                if(saveids[i]==article_id && !hasid) hasid = true;
                saveid += (saveid=='' ? saveids[i] : ','+saveids[i]);
                j++;
                if(j==20 && hasid) break;
                if(j==19 && !hasid) break;
            }
        }

        if(hasid)
        {
            $('#zanbox').addClass('zanbox-selected');
        }

    }

});

window.onload = function() {

    $('#sharediv').removeClass('bdshare-button-style1-32');
    $('#sharediv').attr('class','bdsharebuttonbox');
};
</script>
<script type="text/javascript" src="http://cdn.cocimg.com/cms/templets/js/shCore.js"></script>
    <script type="text/javascript" src="http://cdn.cocimg.com/cms/templets/js/brush.js"></script>
    <link type="text/css" rel="stylesheet" href="http://cdn.cocimg.com/cms/templets/style/shCoreDefault.css"/>
    <script type="text/javascript">
    //SyntaxHighlighter.config.clipboardSwf = '/js/scripts/clipboard.swf';

    SyntaxHighlighter.all();</script>


<script>window._bd_share_config={"common":{"bdSnsKey":{},"bdText":"深入研究 Runloop 与线程保活 - 来自CocoaChina苹果开发中文站@CocoaChina","bdMini":"2","bdMiniList":false,"bdPic":"http://api.cocoachina.com/uploads/160727/1bca2f63de7ff2c3ff3f414da8de7ff7.jpg","bdStyle":"1","bdSize":"32"},"share":{}};with(document)0[(getElementsByTagName('head')[0]||body).appendChild(createElement('script')).src='http://bdimg.share.baidu.com/static/api//js/share.js?v=89860593.js?cdnversion='+~(-new Date()/36e5)];</script>
<!--./cont-->

<!-- 回复评论框模板 -->
<script type="text/javascript" src="http://cdn.cocimg.com/cms/templets/js/comment.js?v=20151019"></script>
<script type="text/javascript" src="http://cdn.cocimg.com/cms/templets/js/mustache.min.js"></script>
<!-- 回复评论框模板 -->
<script id="reply_comment_template" type="x-tmpl-mustache">
    <div id="sub-reply-comment" class="reply-comment">
        <textarea name="subcomment" id="sub-reply-txt" cols="30" rows="10" placeholder="你怎么看？快来评论一下吧！"></textarea>
        <input type="hidden" id="sub-reply-id" value="{{commentid}}" />
        <input type="hidden" id="sub-reply-userid" value="{{parentuserid}}" />
        <input type="hidden" id="sub-reply-username" value="{{parentusername}}" />
        <a type="button" id="sub_comment_bt" class="submitbt">提交</a>
        <div class="clearfloat"></div>
        <div class="unlogin-mask">
            <div>您还没有登录！请<a href="/bbs/login.php">登录</a>或<a href="/bbs/register.php">注册</a></div>
        </div>
    </div>
</script>

<!-- 评论模板 -->
<script id="main_comment_template" type="x-tmpl-mustache">
    <div class="glab-comment" rel-data-id="{{comment_id}}" rel-user-id="{{user_id}}" rel-user-name="{{user_name}}" rel-layer="1">
        <div class="comment-left">
            <a href="/bbs/u.php?action=feed&uid={{user_id}}" target="_blank"><img src="{{user_icon_url}}" height="43" width="43" alt=""></a>
        </div>
        <div class="comment-right">
            <div class="comment-author"><a href="/bbs/u.php?action=feed&uid={{user_id}}" target="_blank">{{user_name}}</a><span class="puptime">{{create_time}}</span></div>
            <div class="comment-content">{{&content}}</div>
            <div class="comment-buttons">
                <span class="ding">                    
                    <img src="http://cdn.cocimg.com/cms/templets/images/zan-default.png" height="14" width="14" alt="" class="default" style="{{#has_supported}} display:none; {{/has_supported}}">
                    <img src="http://cdn.cocimg.com/cms/templets/images/zan-active.png" height="14" width="14" alt="" class="actived" style="{{^has_supported}} display:none; {{/has_supported}}">
                    <span class="zancnt">{{support}}</span>
                </span>
                <span class="cai">
                    <img src="http://cdn.cocimg.com/cms/templets/images/cai-default.png" height="14" width="14" alt="" class="default" style="{{#has_reported}} display:none; {{/has_reported}}">
                    <img src="http://cdn.cocimg.com/cms/templets/images/cai-active.png" height="14" width="14" alt="" class="actived" style="{{^has_reported}} display:none; {{/has_reported}}">
                    <span class="zancnt">{{report}}</span>
                </span>
                <span class="replybt">回复</span>
            </div>
        </div>
            {{#sub}}
                <div class="glab-comment glab-sub-comment" rel-data-id="{{comment_id}}" rel-user-id="{{user_id}}" rel-user-name="{{user_name}}" rel-layer="2">
                    <div class="comment-left">
                        <a href="/bbs/u.php?action=feed&uid={{user_id}}" target="_blank"><img src="{{user_icon_url}}" height="43" width="43" alt=""></a>
                    </div>
                    <div class="comment-right">
                        <div class="comment-author"><a href="/bbs/u.php?action=feed&uid={{user_id}}" target="_blank">{{user_name}}</a><span class="puptime">{{create_time}}</span></div>
                        <div class="comment-content"><a href="/bbs/u.php?action=feed&uid={{parent_user_id}}" target="_blank">回复：{{parent}}</a>{{&content}}</div>
                        <div class="comment-buttons">
                            <span class="ding">
                                <img src="http://cdn.cocimg.com/cms/templets/images/zan-default.png" height="14" width="14" alt="" class="default" style="{{#has_supported}} display:none; {{/has_supported}}">
                                <img src="http://cdn.cocimg.com/cms/templets/images/zan-active.png" height="14" width="14" alt="" class="actived" style="{{^has_supported}} display:none; {{/has_supported}}">
                                <span class="zancnt">{{support}}</span>
                            </span>
                            <span class="cai">
                                <img src="http://cdn.cocimg.com/cms/templets/images/cai-default.png" height="14" width="14" alt="" class="default" style="{{#has_reported}} display:none; {{/has_reported}}">
                                <img src="http://cdn.cocimg.com/cms/templets/images/cai-active.png" height="14" width="14" alt="" class="actived" style="{{^has_reported}} display:none; {{/has_reported}}">
                                <span class="zancnt">{{report}}</span>
                            </span>
                            <span class="replybt">回复</span>
                        </div>          
                    </div>
                </div>
            {{/sub}}         
    </div>  
</script>


<script id="sub_comment_template" type="x-tmpl-mustache">
    <div class="glab-comment glab-sub-comment" rel-data-id="{{comment_id}}" rel-user-id="{{user_id}}" rel-user-name="{{user_name}}" rel-layer="2">
        <div class="comment-left">
            <a href="/bbs/u.php?action=feed&uid={{user_id}}" target="_blank"><img src="{{user_icon_url}}" height="43" width="43" alt=""></a>
        </div>
        <div class="comment-right">
            <div class="comment-author"><a href="/bbs/u.php?action=feed&uid={{user_id}}" target="_blank">{{user_name}}</a><span class="puptime">{{create_time}}</span></div>
            <div class="comment-content"><a href="/bbs/u.php?action=feed&uid={{parent_user_id}}" target="_blank">回复：{{parent}}</a>{{&content}}</div>
            <div class="comment-buttons">
                <span class="ding">
                    <img src="http://cdn.cocimg.com/cms/templets/images/zan-default.png" height="14" width="14" alt="" class="default" style="{{#has_supported}} display:none; {{/has_supported}}">
                    <img src="http://cdn.cocimg.com/cms/templets/images/zan-active.png" height="14" width="14" alt="" class="actived" style="{{^has_supported}} display:none; {{/has_supported}}">
                    <span class="zancnt">{{support}}</span>
                </span>
                <span class="cai">
                    <img src="http://cdn.cocimg.com/cms/templets/images/cai-default.png" height="14" width="14" alt="" class="default" style="{{#has_reported}} display:none; {{/has_reported}}">
                    <img src="http://cdn.cocimg.com/cms/templets/images/cai-active.png" height="14" width="14" alt="" class="actived" style="{{^has_reported}} display:none; {{/has_reported}}">
                    <span class="zancnt">{{report}}</span>
                </span>
                <span class="replybt">回复</span>
            </div>          
        </div>
    </div>
</script>


<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-68948817-1', 'auto', {'name': 'newTracker'});
  ga('newTracker.send', 'pageview');

</script>
</div>    
<!--右侧浮动-->
    <div class="r-bar">
        <img src="http://cdn.cocimg.com/cms/templets/images/coco_weixin.png" class="weixin">
    	<a href="http://weibo.com/cocoachina" target="_blank" title="" class="weibo">sina</a>
        <a href="javascript:void(0);" target="_blank" title="" class="feedback">weixin</a>
        <a href="mailto:support@cocoachina.com" title="" class="kefu">mail</a>
        <a href="javascript:;" title="回到顶部" class="returntop">回到顶部</a>
    </div>

<script language="javascript" src="/cms/bbsauth_plus2015.php" type="text/javascript"></script>
<script type="text/javascript">
$(function(){
    if (typeof(_user_info_html) != 'undefined' && _user_info_html)
    {
        $('.nav-user-login').hide();
        $('.nav-user-name').html(_user_info_html).show();
    }

	var type = $('#type_name').html();
	if(type)
	{	
		switch( type )
		{
			case 'iOS开发':
				$('#ios').addClass('light');
				break;
			case 'AppStore研究':
				$('#app_store').addClass('light');
				break;
			case 'Mac开发':
				$('#mac').addClass('light');
				break;
			case '产品设计':
				$('#design').addClass('light');
				break;
			case 'Cocos引擎':
				$('#cocos').addClass('light');
				break;
			case 'WebApp':
				$('#webapp').addClass('light');
				break;
			case 'Swift':
				$('#swift').addClass('light');
				break;
			case '游戏开发':
				$('#game').addClass('light');
				break;
			case '苹果相关':
				$('#apple').addClass('light');
				break;
			case '营销推广':
				$('#market').addClass('light');
				break;		
			case '业界动态':
				$('#industry').addClass('light');
				break;	
			case '程序人生':
				$('#programmer').addClass('light');
				break;																																
				
		}
	}
});
</script>
<!--./cont-->
<script type="text/javascript" src="/cms/templets/js/index.js?20111546514"></script>
<script src="http://cdn.cocimg.com/cms/templets/js/jquery.colorbox-min.js" type="text/javascript"></script>
<script language="javascript" src="/cms/bbsauth_new2015.php"  type="text/javascript"></script>
<script src="/cms/templets/js/script.js?t=2045479411" type="text/javascript"></script>
<script type="text/javascript">
			jQuery(function()
			{
				//更改文章来源
				var source = $('#source a').html();								
				if( source == '未知' )
					jQuery('#source a').html('CocoaChina');
				if( jQuery('#source a').attr('href')=='')
					jQuery('#source').html('来源：'+source);
				
				//获取用户登录的id
//                var content = jQuery('.t-lid').attr("href");
//                var uid = 0;
//                if( content )
//                {
//                    content = content.split('=');
//                    uid = content[1];
//                }
//                var timestamp = new Date().getTime();	
//                var uid2 = parseInt(uid);
//                if(uid2==uid && uid2!=0){
//                    uid = uid<<2;			
//                    uid = uid+''+timestamp;
//                }else{
//                    uid = "";
//                }
//                var oIframe=document.getElementById("comiframe");
//                if(oIframe){
//                    var oUrl=window.location.href;
//                    oUrl = encodeURIComponent(oUrl);
//
//                    oIframe.src="/comment.php?url="+oUrl+"&uid="+uid;
//                }

				
			});
		</script>
<!--footer-->
<div class="footer clearfix">
    <div class="footer-b">
        <div>
            <a href="/aboutus/index.html" target="_blank">关于我们</a>
            <a href="/contactus/index.html" target="_blank">商务合作</a>
            <a href="mailto:support@cocoachina.com" target="_blank">联系我们</a>
            <a href="/corps" target="_blank">合作伙伴</a>
        </div>
        <p class="footer-b-p">北京触控科技有限公司版权所有</p>
        <p>
            &copy;2016 Chukong Technologies,Inc.
            <i>
                <a href="http://www.miibeian.gov.cn/" target="_blank">京ICP备 11006519号 　京ICP证 100954号</a> 京公网安备11010502020289 
                <a href="http://182.131.21.137/ccnt-apply/admin/business/preview/business-preview!lookUrlRFID.action?main_id=A580B24B1AE846CE9DA949026D63939F" target="_blank">
                    <img src="http://cdn.cocimg.com/cms/templets/images/jingwen.png" style="margin: -10px auto -14px -10px;" />
                    京网文[2012]0426-138号
                </a>
            </i>
        </p>
    </div>
</div>
<!--./footer-->
<script language="javascript" type="text/javascript" src="/cms/include/dedeajax2-min.js"></script>
<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "//hm.baidu.com/hm.js?88399e4c4cf744609c4fc8c3a8935691";
  var s = document.getElementsByTagName("script")[0]; 
  s.parentNode.insertBefore(hm, s);
})();
</script>

<script type="text/javascript">

  var _gaq = _gaq || [];
  _gaq.push(['_setAccount', 'UA-16940817-2']);
  _gaq.push(['_trackPageview']);

  (function() {
    var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
    ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
  })();
  
 
function search_check()
{
	var search_keyword = $('#search_keyword').val();

	if( search_keyword.length < 2 )
	{
		alert("搜索关键字不能少于2个字节，请重新输入。");
	}
	else
	{
		$('#searchform').submit();
	}
}

</script>
<script type="text/javascript">
_acK({aid:116313,format:0,mode:1,gid:1,serverbaseurl:"afp.csbew.com/"});
</script>

</body>
</html>
