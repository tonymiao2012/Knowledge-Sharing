### 1. Component，VUE中的一个重要概念

接触了VUE之后，第一感觉是上手容易。但是有很多坑等着去填。最近做了些组件出来，有了些心得。我将结合着一些文字资料来写一篇分享。

现在VUE.js相关的开源项目数不胜数，比如这个链接：[VUE开源项目库汇总](http://www.cnblogs.com/opendigg/p/6513510.html)总结了一些高质量的库。其中拿iView的一段代码举个小例子：

<pre><code><span></span><span class="p">&lt;</span><span class="nt">i-input</span> <span class="na">type</span><span class="o">=</span><span class="s">"text"</span> <span class="na">:value</span><span class="err">.</span><span class="na">sync</span><span class="o">=</span><span class="s">"formInline.user"</span> <span class="na">placeholder</span><span class="o">=</span><span class="s">"Username"</span><span class="p">&gt;</span>
     <span class="p">&lt;</span><span class="nt">Icon</span> <span class="na">type</span><span class="o">=</span><span class="s">"ios-person-outline"</span> <span class="na">slot</span><span class="o">=</span><span class="s">"prepend"</span><span class="p">&gt;&lt;/</span><span class="nt">Icon</span><span class="p">&gt;</span>
<span class="p">&lt;/</span><span class="nt">i-input</span><span class="p">&gt;</span>
</code></pre>
会得到一个带有ICON和默认输入的控件效果。

### 附录：那些需要注意的大坑和技巧

#### 关于vuejs大小写，camcase等

在使用vueify时候，需要import一个组件的配置对象，这时建议全部使用首字母大写的命名方式，如：
<pre><code>
   import MyComponent from './my-component'

   export default {
     components: {
       MyComponent // es2015 shorhand
     }
   }
   //然后在template中使用-代替非首单词大写字母:
   &lt;my-component>&lt;/my-component>
</code></pre>

#### Dynamic components with prop data

如果一个挂载点需要不同的component，这时dynamic component就很合适了。比如下面的例子代码：

<pre><code>
&lt;!-- html --&gt;
&lt;div id="app"&gt;
  by dynamic Component:
  &lt;component
    v-for="item in items"
    :is="item.component"  //这里根据item.component来决定渲染哪类组件
    :opts="item.options"&gt; //同时通过opts传入要渲染组件的props
  &lt;/component&gt;
</code></pre>

<pre><code>
Vue.component('node', {
  template: "&lt;div&gt;node: {{ opts }}&lt;/div&gt;",
  props: ['opts']
});

Vue.component('node2', {
  template: "&lt;div&gt;node2: {{ opts }}&lt;/div&gt;",
  props: ['opts']
});

new Vue({
  el: '#app',
  data() {
    return {
      items: [{
        component: "node",  //node节点类型
        options: 'node节点数据'
      }, {
        component: "node2", //node2节点类型
        options: 'node2节点数据'
      }]
    };
  }
  methods: {
    addItem() {
      this.items.push(this.newItem);
      this.newItem = {
        component: "",
        options: ""
      }
    }
  }
});
</code></pre>

实际运行结果： [Fiddle](https://jsfiddle.net/matiascx/qn29r3vt/)

