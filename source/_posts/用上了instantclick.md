---
title: 用上了instantclick
date: 2015-03-06 01:26:10
tags:
  - website
---

一开始是看到谭总博客里面那纵享丝滑的像是ajax的加载

（还有上面那个神奇的进度条）

然后就发现了这个神奇的instantclick

利用pjax，在鼠标放到a上的时候开始加载页面，当你点下鼠标的时候页面已经加载完毕了，这时只要替换页面的title和body，纵享丝滑~

但是评论框比较蛋疼，因为是内嵌的script，在每次加载页面之后它并没有重置重新加载

解决的办法是用上instantclick提供的change事件

```html
<script data-no-instant>
var disqus_shortname = 'hcyue';
InstantClick.on('change', function(isInitialLoad) {
    if (!isInitialLoad) {
        // update Disqus thread
        if (document.querySelector("#disqus_thread")) {
            if (typeof DISQUS === 'undefined') {
                var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
                dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
                (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
            } else {
                DISQUS.reset({
                  reload: true,
                  config: function () {
                    this.page.identifier = window.location.pathname;
                    this.page.url = window.location.toString();
                  }
                });
            }
        }

    }
});
InstantClick.init();
</script>
```

判断在内容变化时页面存不存在评论框，决定是否加载远程脚本。

改天研究一下这个instantclick，虽然纵享丝滑但是似乎毛病略多