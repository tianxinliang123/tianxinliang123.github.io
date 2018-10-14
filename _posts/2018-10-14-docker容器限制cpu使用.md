<div id="mainContent">
	<div class="forFlow">
		
<div id="post_detail">
<!--done-->
<div id="topics">
	<div class="post">
		<h1 class="postTitle">
			<a id="cb_post_title_url" class="postTitle2" href="https://www.cnblogs.com/sparkdev/p/8052522.html">Docker: 限制容器可用的 CPU</a>
		</h1>
		<div class="clear"></div>
		<div class="postBody">
			<div id="cnblogs_post_body" class="blogpost-body"><p><span style="font-family: Microsoft YaHei; font-size: 15px">默认情况下容器可以使用的主机 CPU 资源是不受限制的。和内存资源的使用一样，如果不对容器可以使用的 CPU 资源进行限制，一旦发生容器内程序异常使用 CPU 的情况，很可能把整个主机的 CPU 资源耗尽，从而导致更大的灾难。本文将介绍如何限制容器可以使用的 CPU 资源。</span><br><span style="font-family: Microsoft YaHei; font-size: 15px">本文的 demo 中会继续使用《<a href="http://www.cnblogs.com/sparkdev/p/8032330.html" target="_blank">Docker: 限制容器可用的内存</a>》一文中创建的 docker 镜像 u-stress 进行压力测试，文中就不再过多的解释了。</span></p>
<h1><span style="font-family: Microsoft YaHei; font-size: 18pt">限制可用的 CPU 个数</span></h1>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">在 docker 1.13 及更高的版本上，能够很容易的限制容器可以使用的主机 CPU 个数。只需要通过 --cpus 选项指定容器可以使用的 CPU 个数就可以了，并且还可以指定如 1.5 之类的小数。接下来我们在一台有四个 CPU 且负载很低的主机上进行 demo 演示：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217163410546-1515553124.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">通过下面的命令创建容器，--cpus=2 表示容器最多可以使用主机上两个 CPU：</span></p>
<div class="cnblogs_code">
<pre>$ docker run -it --<span style="color: #0000ff">rm</span> --cpus=<span style="color: #800080">2</span> u-stress:latest /bin/bash</pre>
</div>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">然后由 stress 命令创建四个繁忙的进程消耗 CPU 资源：</span></p>
<div class="cnblogs_code">
<pre># stress -c <span style="color: #800080">4</span></pre>
</div>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">我们先来看看 docker stats 命令的输出：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217163517202-62045189.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">容器 CPU 的负载为 200%，它的含义为单个 CPU 负载的两倍。我们也可以把它理解为有两颗 CPU 在 100% 的为它工作。</span><br><span style="font-family: Microsoft YaHei; font-size: 15px">再让我们通过 top 命令看看主机 CPU 的真实负载情况：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217163457514-1473983991.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">哈哈，有点大跌眼镜！实际的情况并不是两个 CPU 负载 100%，而另外两个负载 0%。四个 CPU 的负载都是 50%，加起来容器消耗的 CPU 总量就是两个 CPU 100% 的负载。</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">看来对于进程来说是没有 CPU 个数这一概念的，内核只能通过进程消耗的 CPU 时间片来统计出进程占用 CPU 的百分比。这也是我们看到的各种工具中都使用百分比来说明 CPU 使用率的原因。</span><br><span style="font-family: Microsoft YaHei; font-size: 15px">严谨起见，我们看看 docker 的官方文档中是如何解释 --cpus 选项的：</span><br><a href="https://docs.docker.com/engine/admin/resource_constraints/#configure-the-default-cfs-scheduler" target="_blank"><span style="font-family: Microsoft YaHei; font-size: 15px"><strong>Specify how much of the available CPU resources a container can use</strong>.</span></a><br><span style="font-family: Microsoft YaHei; font-size: 15px">果然，人家用的是 "how much"，不可数的！并且 --cpus 选项支持设为小数也从侧面说明了对 CPU 的计量只能是百分比。</span><br><span style="font-family: Microsoft YaHei; font-size: 15px">看来笔者在本文中写的 "CPU 个数" 都是不准确的。既然不准确，为什么还要用？当然是为了容易理解。况且笔者认为在 --cpus 选项的上下文中理解为 "CPU 个数" 并没有问题(有兴趣的同学可以读读 <a href="https://github.com/moby/moby/issues/27921" target="_blank">--cpus 选项的由来</a>，人家的初衷也是要表示 CPU 个数的)。</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">虽然 --cpus 选项用起来很爽，但它毕竟是 1.13 才开始支持的。对于更早的版本完成同样的功能我们需要配合使用两个选项：--cpu-period 和 --cpu-quota(1.13 及之后的版本仍然支持这两个选项)。下面的命令实现相同的结果：</span></p>
<div class="cnblogs_code">
<pre>$ docker run -it --<span style="color: #0000ff">rm</span> --cpu-period=<span style="color: #800080">100000</span> --cpu-quota=<span style="color: #800080">200000</span> u-stress:latest /bin/bash</pre>
</div>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">这样的配置选项是不是让人很傻眼呀！100000 是什么？200000 又是什么？ 它们的单位是微秒，100000 表示 100 毫秒，200000 表示 200 毫秒。它们在这里的含义是：在每 100 毫秒的时间里，运行进程使用的 CPU 时间最多为 200 毫秒(需要两个 CPU 各执行 100 毫秒)。要想彻底搞明白这两个选项的同学可以参考：<a href="https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt" target="_blank">CFS BandWith Control</a>。我们要知道这两个选项才是事实的真相，但是真相往往很残忍！还好 --cpus 选项成功的解救了我们，其实它就是包装了 --cpu-period 和 --cpu-quota。</span></p>
<h1><span style="font-family: Microsoft YaHei; font-size: 18pt">指定固定的 CPU</span></h1>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">通过 --cpus 选项我们无法让容器始终在一个或某几个 CPU 上运行，但是通过 --cpuset-cpus 选项却可以做到！这是非常有意义的，因为现在的多核系统中每个核心都有自己的缓存，如果频繁的调度进程在不同的核心上执行势必会带来缓存失效等开销。下面我们就演示如何设置容器使用固定的 CPU，下面的命令为容器设置了 --cpuset-cpus 选项，指定运行容器的 CPU 编号为 1：</span></p>
<div class="cnblogs_code">
<pre>$ docker run -it --<span style="color: #0000ff">rm</span> --cpuset-cpus=<span style="color: #800000">"</span><span style="color: #800000">1</span><span style="color: #800000">"</span> u-stress:latest /bin/bash</pre>
</div>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">再启动压力测试命令：</span></p>
<div class="cnblogs_code">
<pre># stress -c <span style="color: #800080">4</span></pre>
</div>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">然后查看主机 CPU 的负载情况：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217163742921-1627559356.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">这次只有 Cpu1 达到了 100%，其它的 CPU 并未被容器使用。我们还可以反复的执行 stress -c 4 命令，但是始终都是 Cpu1 在干活。</span><br><span style="font-family: Microsoft YaHei; font-size: 15px">再看看容器的 CPU 负载，<span style="font-family: Microsoft YaHei; font-size: 15px">也是只有 100%</span>：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217163808468-652259868.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">--cpuset-cpus 选项还可以一次指定多个 CPU：</span></p>
<div class="cnblogs_code">
<pre>$ docker run -it --<span style="color: #0000ff">rm</span> --cpuset-cpus=<span style="color: #800000">"</span><span style="color: #800000">1,3</span><span style="color: #800000">"</span> u-stress:latest /bin/bash</pre>
</div>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">这次我们指定了 1，3 两个 CPU，运行 stress -c 4 命令，然后检查主机的 CPU 负载：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217164009218-1499882618.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">Cpu1 和 Cpu3 的负载都达到了 100%。</span><br><span style="font-family: Microsoft YaHei; font-size: 15px">容器的 CPU 负载也达到了 200%：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217164032296-1942575599.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">--cpuset-cpus 选项的一个缺点是必须指定 CPU 在操作系统中的编号，这对于动态调度的环境(无法预测容器会在哪些主机上运行，只能通过程序动态的检测系统中的 CPU 编号，并生成 docker run 命令)会带来一些不便。</span></p>
<h1><span style="font-family: Microsoft YaHei; font-size: 18pt">设置使用 CPU 的权重</span></h1>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">当 CPU 资源充足时，设置 CPU 的权重是没有意义的。只有在容器争用 CPU 资源的情况下， CPU 的权重才能让不同的容器分到不同的 CPU 用量。--cpu-shares 选项用来设置 CPU 权重，它的默认值为 1024。我们可以把它设置为 2 表示很低的权重，但是设置为 0 表示使用默认值 1024。</span><br><span style="font-family: Microsoft YaHei; font-size: 15px">下面我们分别运行两个容器，指定它们都使用 Cpu0，并分别设置 --cpu-shares 为 512 和 1024：</span></p>
<div class="cnblogs_code">
<pre>$ docker run -it --<span style="color: #0000ff">rm</span> --cpuset-cpus=<span style="color: #800000">"</span><span style="color: #800000">0</span><span style="color: #800000">"</span> --cpu-shares=<span style="color: #800080">512</span> u-stress:latest /bin/<span style="color: #000000">bash
$ docker run </span>-it --<span style="color: #0000ff">rm</span> --cpuset-cpus=<span style="color: #800000">"</span><span style="color: #800000">0</span><span style="color: #800000">"</span> --cpu-shares=<span style="color: #800080">1024</span> u-stress:latest /bin/bash</pre>
</div>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">在两个容器中都运行 stress -c 4 命令。</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">此时主机 Cpu0 的负载为 100%：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217164142218-723091322.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">容器中 CPU 的负载为：</span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px"><img src="https://images2017.cnblogs.com/blog/952033/201712/952033-20171217164255702-1003240099.png" alt=""></span></p>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">两个容器分享一个 CPU，所以总量应该是 100%。具体每个容器分得的负载则取决于 --cpu-shares 选项的设置！我们的设置分别是 512 和 1024，则它们分得的比例为 1:2。在本例中如果想让两个容器各占 50%，只要把 <span style="font-family: Microsoft YaHei; font-size: 15px">--cpu-shares 选项设为相同的值就可以了。</span><br></span></p>
<h1><span style="font-family: Microsoft YaHei; font-size: 18pt">总结</span></h1>
<p><span style="font-family: Microsoft YaHei; font-size: 15px">相比限制容器用的内存，限制 CPU 的选项要简洁很多。但是简洁绝对不是简单，大多数把复杂东西整简单的过程都会丢失细节或是模糊一些概念，比如从 --cpu-period 和 --cpu-quota 选项到 --cpus 选项的进化。对于使用者来说这当然是好事，可以减缓我们的学习曲线，快速入手。</span></p></div><div id="MySignature" style="display: block;"><div>作者：<a href="http://www.cnblogs.com/sparkdev/" target="_blank">sparkdev</a></div>
<div>出处：<a href="http://www.cnblogs.com/sparkdev/" target="_blank">http://www.cnblogs.com/sparkdev/</a></div>
<div>本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文连接，否则保留追究法律责任的权利。</div></div>
<div class="clear"></div>
<div id="blog_post_info_block">
<div id="BlogPostCategory">分类: <a href="https://www.cnblogs.com/sparkdev/category/927855.html" target="_blank">Docker</a>,<a href="https://www.cnblogs.com/sparkdev/category/834899.html" target="_blank">Linux</a></div>
<div id="EntryTag">标签: <a href="https://www.cnblogs.com/sparkdev/tag/docker/">docker</a>, <a href="https://www.cnblogs.com/sparkdev/tag/Linux/">Linux</a></div>
<div id="blog_post_info"><div id="green_channel">
        <a href="javascript:void(0);" id="green_channel_digg" onclick="DiggIt(8052522,cb_blogId,1);green_channel_success(this,'谢谢推荐！');">好文要顶</a>
            <a id="green_channel_follow" onclick="follow('636ea2f4-ca14-e611-9fc1-ac853d9f53cc');" href="javascript:void(0);">关注我</a>
    <a id="green_channel_favorite" onclick="AddToWz(cb_entryId);return false;" href="javascript:void(0);">收藏该文</a>
    <a id="green_channel_weibo" href="javascript:void(0);" title="分享至新浪微博" onclick="ShareToTsina()"><img src="//common.cnblogs.com/images/icon_weibo_24.png" alt=""></a>
    <a id="green_channel_wechat" href="javascript:void(0);" title="分享至微信" onclick="shareOnWechat()"><img src="//common.cnblogs.com/images/wechat.png" alt=""></a>
</div>
<div id="author_profile">
    <div id="author_profile_info" class="author_profile_info">
            <a href="http://home.cnblogs.com/u/sparkdev/" target="_blank"><img src="//pic.cnblogs.com/face/952033/20160916141655.png" class="author_avatar" alt=""></a>
        <div id="author_profile_detail" class="author_profile_info">
            <a href="http://home.cnblogs.com/u/sparkdev/">sparkdev</a><br>
            <a href="http://home.cnblogs.com/u/sparkdev/followees">关注 - 23</a><br>
            <a href="http://home.cnblogs.com/u/sparkdev/followers">粉丝 - 475</a>
        </div>
    </div>
    <div class="clear"></div>
    <div id="author_profile_honor">荣誉：<a href="http://www.cnblogs.com/expert/" target="_blank">推荐博客</a></div>
    <div id="author_profile_follow">
                <a href="javascript:void(0);" onclick="follow('636ea2f4-ca14-e611-9fc1-ac853d9f53cc');return false;">+加关注</a>
    </div>
</div>
<div id="div_digg">
    <div class="diggit" onclick="votePost(8052522,'Digg')">
        <span class="diggnum" id="digg_count">19</span>
    </div>
    <div class="buryit" onclick="votePost(8052522,'Bury')">
        <span class="burynum" id="bury_count">0</span>
    </div>
    <div class="clear"></div>
    <div class="diggword" id="digg_tips">
    </div>
</div>
<script type="text/javascript">
    currentDiggType = 0;
</script></div>
<div class="clear"></div>
<div id="post_next_prev"><a href="https://www.cnblogs.com/sparkdev/p/8032330.html" class="p_n_p_prefix">« </a> 上一篇：<a href="https://www.cnblogs.com/sparkdev/p/8032330.html" title="发布于2017-12-14 07:46">Docker: 限制容器可用的内存</a><br><a href="https://www.cnblogs.com/sparkdev/p/8108921.html" class="p_n_p_prefix">» </a> 下一篇：<a href="https://www.cnblogs.com/sparkdev/p/8108921.html" title="发布于2017-12-26 07:54">Azure: 给 ubuntu 虚机挂载数据盘</a><br></div>
</div>


		</div>
		<div class="postDesc">posted @ <span id="post-date">2017-12-19 08:32</span> <a href="https://www.cnblogs.com/sparkdev/">sparkdev</a> 阅读(<span id="post_view_count">37917</span>) 评论(<span id="post_comment_count">8</span>)  <a href="https://i.cnblogs.com/EditPosts.aspx?postid=8052522" rel="nofollow">编辑</a> <a href="#" onclick="AddToWz(8052522);return false;">收藏</a></div>
	</div>
	<script type="text/javascript">var allowComments=true,cb_blogId=287522,cb_entryId=8052522,cb_blogApp=currentBlogApp,cb_blogUserGuid='636ea2f4-ca14-e611-9fc1-ac853d9f53cc',cb_entryCreatedDate='2017/12/19 8:32:00';loadViewCount(cb_entryId);var cb_postType=1;</script>
	
</div><!--end: topics 文章、评论容器-->
</div><a name="!comments"></a><div id="blog-comments-placeholder"><div id="comments_pager_top"></div>
<br>
<div class="feedback_area_title">评论列表</div>
<div class="feedbackNoItems"></div>	

		<div class="feedbackItem">
			<div class="feedbackListSubtitle">
				<div class="feedbackManage">
					&nbsp;&nbsp;<span class="comment_actions"></span>
				</div>
				<a href="#3868898" class="layer">#1楼</a><a name="3868898" id="comment_anchor_3868898"></a>  <span class="comment_date">2017-12-19 08:58</span> <a id="a_comment_author_3868898" href="http://home.cnblogs.com/u/1274995/" target="_blank">忧伤的猫</a> <a href="http://msg.cnblogs.com/send/%E5%BF%A7%E4%BC%A4%E7%9A%84%E7%8C%AB" title="发送站内短消息" class="sendMsg2This">&nbsp;</a>
			</div>
			<div class="feedbackCon">
				<div id="comment_body_3868898" class="blog_comment_body">很实用，楼主也讲的很清楚，收藏了！</div><div class="comment_vote"><a href="javascript:void(0);" class="comment_digg" onclick="return voteComment(3868898,'Digg',this)">支持(0)</a><a href="javascript:void(0);" class="comment_bury" onclick="return voteComment(3868898,'Bury',this)">反对(0)</a></div>
			</div>
		</div>
	
		<div class="feedbackItem">
			<div class="feedbackListSubtitle">
				<div class="feedbackManage">
					&nbsp;&nbsp;<span class="comment_actions"></span>
				</div>
				<a href="#3869131" class="layer">#2楼</a><a name="3869131" id="comment_anchor_3869131"></a>  <span class="comment_date">2017-12-19 12:14</span> <a id="a_comment_author_3869131" href="http://home.cnblogs.com/u/1076647/" target="_blank">一百斤</a> <a href="http://msg.cnblogs.com/send/%E4%B8%80%E7%99%BE%E6%96%A4" title="发送站内短消息" class="sendMsg2This">&nbsp;</a>
			</div>
			<div class="feedbackCon">
				<div id="comment_body_3869131" class="blog_comment_body">感觉 cpus 就应该表示 CPU 的个数，这样理解起来就简单多了(对于刨根问底的情况除外)。</div><div class="comment_vote"><a href="javascript:void(0);" class="comment_digg" onclick="return voteComment(3869131,'Digg',this)">支持(0)</a><a href="javascript:void(0);" class="comment_bury" onclick="return voteComment(3869131,'Bury',this)">反对(0)</a></div>
			</div>
		</div>
	
		<div class="feedbackItem">
			<div class="feedbackListSubtitle">
				<div class="feedbackManage">
					&nbsp;&nbsp;<span class="comment_actions"></span>
				</div>
				<a href="#3869592" class="layer">#3楼</a><a name="3869592" id="comment_anchor_3869592"></a>[<span class="louzhu">楼主</span>]  <span class="comment_date">2017-12-19 18:35</span> <a id="a_comment_author_3869592" href="https://www.cnblogs.com/sparkdev/" target="_blank">sparkdev</a> <a href="http://msg.cnblogs.com/send/sparkdev" title="发送站内短消息" class="sendMsg2This">&nbsp;</a>
			</div>
			<div class="feedbackCon">
				<div id="comment_body_3869592" class="blog_comment_body"><a href="#3869131" title="查看所回复的评论" onclick="commentManager.renderComments(0,50,3869131);">@</a>
一百斤<br>如果真正把背后的东西都搞清楚了，那它表示什么也都不重要了。</div><div class="comment_vote"><a href="javascript:void(0);" class="comment_digg" onclick="return voteComment(3869592,'Digg',this)">支持(0)</a><a href="javascript:void(0);" class="comment_bury" onclick="return voteComment(3869592,'Bury',this)">反对(0)</a></div><span id="comment_3869592_avatar" style="display:none;">http://pic.cnblogs.com/face/952033/20160916141655.png</span>
			</div>
		</div>
	
		<div class="feedbackItem">
			<div class="feedbackListSubtitle">
				<div class="feedbackManage">
					&nbsp;&nbsp;<span class="comment_actions"></span>
				</div>
				<a href="#3869640" class="layer">#4楼</a><a name="3869640" id="comment_anchor_3869640"></a>  <span class="comment_date">2017-12-19 19:44</span> <a id="a_comment_author_3869640" href="http://home.cnblogs.com/u/1123467/" target="_blank">日上三竿</a> <a href="http://msg.cnblogs.com/send/%E6%97%A5%E4%B8%8A%E4%B8%89%E7%AB%BF" title="发送站内短消息" class="sendMsg2This">&nbsp;</a>
			</div>
			<div class="feedbackCon">
				<div id="comment_body_3869640" class="blog_comment_body">cpu-shares是怎么实现的，感觉很神奇？？</div><div class="comment_vote"><a href="javascript:void(0);" class="comment_digg" onclick="return voteComment(3869640,'Digg',this)">支持(0)</a><a href="javascript:void(0);" class="comment_bury" onclick="return voteComment(3869640,'Bury',this)">反对(0)</a></div>
			</div>
		</div>
	
		<div class="feedbackItem">
			<div class="feedbackListSubtitle">
				<div class="feedbackManage">
					&nbsp;&nbsp;<span class="comment_actions"></span>
				</div>
				<a href="#3869789" class="layer">#5楼</a><a name="3869789" id="comment_anchor_3869789"></a>[<span class="louzhu">楼主</span>]  <span class="comment_date">2017-12-20 08:47</span> <a id="a_comment_author_3869789" href="https://www.cnblogs.com/sparkdev/" target="_blank">sparkdev</a> <a href="http://msg.cnblogs.com/send/sparkdev" title="发送站内短消息" class="sendMsg2This">&nbsp;</a>
			</div>
			<div class="feedbackCon">
				<div id="comment_body_3869789" class="blog_comment_body"><a href="#3869640" title="查看所回复的评论" onclick="commentManager.renderComments(0,50,3869640);">@</a>
日上三竿<br>cgroup</div><div class="comment_vote"><a href="javascript:void(0);" class="comment_digg" onclick="return voteComment(3869789,'Digg',this)">支持(0)</a><a href="javascript:void(0);" class="comment_bury" onclick="return voteComment(3869789,'Bury',this)">反对(0)</a></div><span id="comment_3869789_avatar" style="display:none;">http://pic.cnblogs.com/face/952033/20160916141655.png</span>
			</div>
		</div>
	
		<div class="feedbackItem">
			<div class="feedbackListSubtitle">
				<div class="feedbackManage">
					&nbsp;&nbsp;<span class="comment_actions"></span>
				</div>
				<a href="#3870002" class="layer">#6楼</a><a name="3870002" id="comment_anchor_3870002"></a>  <span class="comment_date">2017-12-20 12:22</span> <a id="a_comment_author_3870002" href="http://home.cnblogs.com/u/1089358/" target="_blank">月在柳梢头</a> <a href="http://msg.cnblogs.com/send/%E6%9C%88%E5%9C%A8%E6%9F%B3%E6%A2%A2%E5%A4%B4" title="发送站内短消息" class="sendMsg2This">&nbsp;</a>
			</div>
			<div class="feedbackCon">
				<div id="comment_body_3870002" class="blog_comment_body">支持支持！</div><div class="comment_vote"><a href="javascript:void(0);" class="comment_digg" onclick="return voteComment(3870002,'Digg',this)">支持(0)</a><a href="javascript:void(0);" class="comment_bury" onclick="return voteComment(3870002,'Bury',this)">反对(0)</a></div>
			</div>
		</div>
	
		<div class="feedbackItem">
			<div class="feedbackListSubtitle">
				<div class="feedbackManage">
					&nbsp;&nbsp;<span class="comment_actions"></span>
				</div>
				<a href="#3871299" class="layer">#7楼</a><a name="3871299" id="comment_anchor_3871299"></a>  <span class="comment_date">2017-12-21 21:52</span> <a id="a_comment_author_3871299" href="http://home.cnblogs.com/u/1080611/" target="_blank">双黄蛋</a> <a href="http://msg.cnblogs.com/send/%E5%8F%8C%E9%BB%84%E8%9B%8B" title="发送站内短消息" class="sendMsg2This">&nbsp;</a>
			</div>
			<div class="feedbackCon">
				<div id="comment_body_3871299" class="blog_comment_body">用起来简单理解起来难啊</div><div class="comment_vote"><a href="javascript:void(0);" class="comment_digg" onclick="return voteComment(3871299,'Digg',this)">支持(0)</a><a href="javascript:void(0);" class="comment_bury" onclick="return voteComment(3871299,'Bury',this)">反对(0)</a></div>
			</div>
		</div>
	
		<div class="feedbackItem">
			<div class="feedbackListSubtitle">
				<div class="feedbackManage">
					&nbsp;&nbsp;<span class="comment_actions"></span>
				</div>
				<a href="#3871351" class="layer">#8楼</a><a name="3871351" id="comment_anchor_3871351"></a>[<span class="louzhu">楼主</span>]<span id="comment-maxId" style="display:none;">3871351</span><span id="comment-maxDate" style="display:none;">2017/12/22 7:20:31</span>  <span class="comment_date">2017-12-22 07:20</span> <a id="a_comment_author_3871351" href="https://www.cnblogs.com/sparkdev/" target="_blank">sparkdev</a> <a href="http://msg.cnblogs.com/send/sparkdev" title="发送站内短消息" class="sendMsg2This">&nbsp;</a>
			</div>
			<div class="feedbackCon">
				<div id="comment_body_3871351" class="blog_comment_body"><a href="#3871299" title="查看所回复的评论" onclick="commentManager.renderComments(0,50,3871299);">@</a>
双黄蛋<br>的确是这样的。</div><div class="comment_vote"><a href="javascript:void(0);" class="comment_digg" onclick="return voteComment(3871351,'Digg',this)">支持(0)</a><a href="javascript:void(0);" class="comment_bury" onclick="return voteComment(3871351,'Bury',this)">反对(0)</a></div><span id="comment_3871351_avatar" style="display:none;">http://pic.cnblogs.com/face/952033/20160916141655.png</span>
			</div>
		</div>
	<div id="comments_pager_bottom"></div></div><script type="text/javascript">var commentManager = new blogCommentManager();commentManager.renderComments(0);</script>
<div id="comment_form" class="commentform">
<a name="commentform"></a>
<div id="divCommentShow"></div>
<div id="comment_nav"><span id="span_refresh_tips"></span><a href="javascript:void(0);" onclick="return RefreshCommentList();" id="lnk_RefreshComments" runat="server" clientidmode="Static">刷新评论</a><a href="#" onclick="return RefreshPage();">刷新页面</a><a href="#top">返回顶部</a></div>
<div id="comment_form_container"><div class="login_tips">注册用户登录后才能发表评论，请 <a rel="nofollow" href="javascript:void(0);" class="underline" onclick="return login('commentform');">登录</a> 或 <a rel="nofollow" href="javascript:void(0);" class="underline" onclick="return register();">注册</a>，<a href="http://www.cnblogs.com">访问</a>网站首页。</div></div>
<div class="ad_text_commentbox" id="ad_text_under_commentbox"></div>
<div id="ad_t2"><a href="http://www.ucancode.com/index.htm" target="_blank">【推荐】超50万VC++源码: 大型组态工控、电力仿真CAD与GIS源码库！</a><br><a href="http://clickc.admaster.com.cn/c/a116493,b2949399,c1705,i0,m101,8a1,8b3,h" target="_blank" onclick="ga('send', 'event', 'Link', 'click', 'T2-华为云')">【推荐】华为云全联接大会 高性能云服务器限时2折</a><br><a href="https://cloud.tencent.com/act/domainsales?fromSource=gwzcw.1351351.1351351.1351351" target="_blank" onclick="ga('send', 'event', 'Link', 'click', 'T2-域名')">【推荐】腾讯云新注册用户域名抢购1元起</a><br></div>
<div id="opt_under_post"></div>
<div id="cnblogs_c1" class="c_ad_block"><a href="https://cloud.tencent.com/act/special/amd?fromSource=gwzcw.1351353.1351353.1351353" target="_blank"><img width="300" height="250" src="https://img2018.cnblogs.com/news/24442/201810/24442-20181008121639348-1607410764.jpg" alt="腾讯云1008" onclick="ga('send', 'event', 'Link', 'click', 'C1');"></a></div>
<div id="under_post_news"><div class="itnews c_ad_block"><b>最新IT新闻</b>:<br> ·  <a href="https://news.cnblogs.com/n/609484/" target="_blank">这些百强县隐藏了中国的另一面</a><br> ·  <a href="https://news.cnblogs.com/n/609483/" target="_blank">微软的智能指环新专利再度曝光 可识别手势和对其施加的压力</a><br> ·  <a href="https://news.cnblogs.com/n/609482/" target="_blank">国内首个原创长效抗艾新药上市 到底有何奥秘？</a><br> ·  <a href="https://news.cnblogs.com/n/609481/" target="_blank">微软员工发公开信反对公司投标美国防部JEDI云项目</a><br> ·  <a href="https://news.cnblogs.com/n/609480/" target="_blank">来电袁炳松：5亿人民币是分水岭，共享充电宝的行业终局会是收并购</a><br>» <a href="http://news.cnblogs.com/" title="IT新闻" target="_blank">更多新闻...</a></div></div>
<script async="async" src="https://www.googletagservices.com/tag/js/gpt.js"></script>
<script>
  var googletag = googletag || {};
  googletag.cmd = googletag.cmd || [];
</script>

<script>
  googletag.cmd.push(function() {
    googletag.defineSlot('/1090369/C2', [468, 60], 'div-gpt-ad-1539008685004-0').addService(googletag.pubads());
    googletag.pubads().enableSingleRequest();
    googletag.enableServices();
  });
</script>
<div id="cnblogs_c2" class="c_ad_block">
    <div id="div-gpt-ad-1539008685004-0" style="height:60px; width:468px;">
    <script>
    if (new Date() >= new Date(2018, 9, 13)) {
        googletag.cmd.push(function() { googletag.display('div-gpt-ad-1539008685004-0'); });
    }
    </script>
    </div>
</div>
<div id="under_post_kb"><div class="itnews c_ad_block" id="kb_block"><b>最新知识库文章</b>:<br><div id="kb_recent"> ·  <a href="https://kb.cnblogs.com/page/606682/" target="_blank">为什么说 Java 程序员必须掌握 Spring Boot ？</a><br> ·  <a href="https://kb.cnblogs.com/page/606645/" target="_blank">在学习中，有一个比掌握知识更重要的能力</a><br> ·  <a href="https://kb.cnblogs.com/page/603663/" target="_blank">如何招到一个靠谱的程序员</a><br> ·  <a href="https://kb.cnblogs.com/page/573614/" target="_blank">一个故事看懂“区块链”</a><br> ·  <a href="https://kb.cnblogs.com/page/603697/" target="_blank">被踢出去的用户</a><br></div>» <a href="https://kb.cnblogs.com/" target="_blank">更多知识库文章...</a></div></div>
<div id="HistoryToday" class="c_ad_block"></div>
<script type="text/javascript">
    fixPostBody();
    setTimeout(function () { incrementViewCount(cb_entryId); }, 50);
    deliverAdT2();
    deliverAdC1();
    deliverAdC2();    
    loadNewsAndKb();
    loadBlogSignature();
    LoadPostInfoBlock(cb_blogId, cb_entryId, cb_blogApp, cb_blogUserGuid);
    GetPrevNextPost(cb_entryId, cb_blogId, cb_entryCreatedDate, cb_postType);
    loadOptUnderPost();
    GetHistoryToday(cb_blogId, cb_blogApp, cb_entryCreatedDate);   
</script>
</div>


	</div><!--end: forFlow -->
	</div>
