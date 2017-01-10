---
layout:     post
title:      "从多说切到Disqus"
subtitle:   "一场由“disqus_shortname”引发的“血案”"
date:       2017-01-10
author:     "ClaireChin"
header-img: "img/post-bg-disqus.jpg"
catalog: true
tags:
    - Disqus
    - 多说
---
>也许对各位看客来说从“多说”切到"Disqus"根本就是a piece of cake的事，或者连“事儿”都算不上。但对于前前后后折腾了几个小时的我来说，
这确实算得上是一场“血案”，或者至少是一个“狠”的教训。

# 多说 or Disqus，it's a question.
------

> * 切换原因
> * 切换过程

![git-logo](https://www.emyspot.com/medias/images/disqus-blog.jpg)

------

## 切换原因

多说能够支持国内的社交网站账号登录，速度快，但在被收购后维护力度低，评论审核机制也不到位。Disqus是国外的评论系统，容易被墙，加载速度也慢，
关联的是国外第三方账号如Twitter，Fackbook之类的，但网上评价还算正面。考虑到多说团队可能说解散就解散，think after twice，
我还是决定把评论系统从多说切换到Disqus。

## 切换过程

整个切换过程其实真的很简单，所以我这种折腾了几个小时的也真的是没谁了。首先在Disqus网站上注册账号，在个人主页的右上角找到setting后点击
"Add Disqus to Site"，接着就是按照网页向导一步步走下去。假如你跟我一样使用了Jekyll搭建博客，那么只要在_config.yml中定义disqus_shortname
并给其赋值，然后在添加评论框的地方添加


    <div id="disqus_thread"></div>
    

以及javascript代码即可，

    <script type="text/javascript">
      /* * * CONFIGURATION VARIABLES * * */
      var disqus_shortname = "{{site.disqus_username}}";
      var disqus_identifier = "{{page.id}}";
      var disqus_url = "{{site.url}}{{page.url}}";
      (function() { // DON'T EDIT BELOW THIS LINE
      var d = document, s = d.createElement('script');
      s.src = '//' + disqus_shortname + '.disqus.com/embed.js';
      s.setAttribute('data-timestamp', +new Date());
      (d.head || d.body).appendChild(s);
      })();
    </script>
  
 这边得把这该死的[disqus_shortname](http://www.jianshu.com)提出来说一说，实际上它就对应你"Configure Disqus for Your Site"页面的
 ![Shortname](/img/post-sample-disqus-shortname.png)
 
 然而我一直把它理解为了![这个](/img/post-sample-wrong_understanding.png)
 
 <font color=#FF4500> 所以做一件事情，如果一开始的出发点或者理解就错了，那真是很要命的！</font>

 
 最后，非常感谢[小兀](https://disqus.com/by/linxiaowu/)的倾力相助。
