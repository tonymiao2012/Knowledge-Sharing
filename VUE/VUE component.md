### 1. Component，VUE中的一个重要概念

接触了VUE之后，第一感觉是上手容易。但是有很多坑等着去填。最近做了些组件出来，有了些心得。我将结合着一些文字资料来写一篇分享。

现在VUE.js相关的开源项目数不胜数，比如这个链接：[VUE开源项目库汇总](http://www.cnblogs.com/opendigg/p/6513510.html)总结了一些高质量的库。其中拿iView的一段代码举个小例子：

<pre><code><span></span><span class="p">&lt;</span><span class="nt">i-input</span> <span class="na">type</span><span class="o">=</span><span class="s">"text"</span> <span class="na">:value</span><span class="err">.</span><span class="na">sync</span><span class="o">=</span><span class="s">"formInline.user"</span> <span class="na">placeholder</span><span class="o">=</span><span class="s">"Username"</span><span class="p">&gt;</span>
     <span class="p">&lt;</span><span class="nt">Icon</span> <span class="na">type</span><span class="o">=</span><span class="s">"ios-person-outline"</span> <span class="na">slot</span><span class="o">=</span><span class="s">"prepend"</span><span class="p">&gt;&lt;/</span><span class="nt">Icon</span><span class="p">&gt;</span>
<span class="p">&lt;/</span><span class="nt">i-input</span><span class="p">&gt;</span>
</code></pre>
会得到一个输入控件效果。