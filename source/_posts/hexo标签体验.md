---
title: hexo标签体验
date: 2017-07-23 01:34:12
tags:
 - hexo
 - tags
---

 ### prologue

 今天搭好了hexo的博客，选了一个风格偏简化的主题。
 那么接下来就来测试下hexo的标签插件的具体效果。

 ### tag test

 hexo的标签插件主要用于在文章中快速插入特定的内容。写法上有点像是jinja这样的模板语言。
 参考hexo文档，其标签插件包括引用、代码高亮和插入外链和图片的功能外，还支持文件代码引入，iframe, gist和youtube视频插入等。

 #### 引用块

 在文章中插入引用块，可以包含作者、来源和标题。

 {% blockquote Yabin Zhang, Yabin Zhang's Blog %}
 引用块测试。

 {% endblockquote %}

 {% blockquote Yabin Zhang %}
 再次引用块测试。

 {% endblockquote %}

 {% blockquote Yab Zhang https://yabzhang.github.io, Test Plugin %}
 Try in English.
 {% endblockquote %}

 引用块标识来源的展示有点问题。

 #### 代码块

 普通代码块
 {% codeblock %}
 alert('Hello, World');
 {% endcodeblock %}

 指定语言
 {% codeblock lang:python %}
 [i for i in range(10)]  # Python
 {% endcodeblock %}

 {% codeblock lang:objc %}
 [rectangle setX: 10 y: 10 width: 20 height: 20];  // objc
 {% endcodeblock %}

 附加说明
 {% codeblock generator lang:python %}
 def generator(seq):
     for item in range(seq):
         yield item
 {% endcodeblock %}

 {% codeblock Array.map lang:js %}
 function test(x, y) {
  return x + y;
 }
 {% endcodeblock %}

 {% codeblock printf lang:c %}
 printf("Hello, World\n");
 {% endcodeblock %}

 指明代码块的语言后可以语法高亮；
 不过附加说明的样式不是很满意。

 #### 反引号代码块

 ```
 funciton print_log(item) {
   console.log(item);
 }
 ```

 ```
 @decorator
 def logic_handler(a, *args, **kwargs):
     pass
 ```

 #### Gist插入

 {% gist d172d3f67336ba1891dce2646cf5dfdd %}

 gist插入的特性挺棒的，但是在gist中包含多个文件时，指定文件名展示失败；

 #### Image

 资源文件引用：
 {% asset_img ayanami.jpg local file %}

 外链图片, 可指定大小:
 {% img http://icons.iconarchive.com/icons/artcore-illustrations/artcore-4/512/github-icon.png 200 200 %}

 #### 外链

 {% link Hexo So Cool https://hexo.io/zh-cn/doc %}

 #### 插入代码文件

  {% include_code ../requests_test.py %}

 代码文件展示失败，又或者我姿势不对。

 #### 插入youtube视频

  昨天看到Linkin Park主唱自杀的消息，感到特别惋惜。
  在我最黑暗的时候，那壮怀激烈的歌声不止一次地给我希望。
  给世界都来了那么多光明，却没一缕光能留得住你。
  一路好走。

  {% youtube kXYiU_JCYtU %}

  ### end

  以上就是对新博客的测试。
  有些地方还有瑕疵，但整体上还比较满意。
