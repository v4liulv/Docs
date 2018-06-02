# HTML 访问本地 Markdown 文件

**目的** 
  我们有一个满文件的 Markdown 文件，需要以HTML的方式展示。

**方法**
使用 HTML 静态文件，引入 jquery 以及 markdown-it 代码库，帮我们的 HTML 拥有处理能力。

**假设**
参考非常简单的开源项目 wxqee/markdown-html-example
> 1. 我们已经有个 readme.md
> 2. 我们已经启动了一个 HTTP 服务器在当前目录

**步骤**

> 1、创建一个 Markdown 转为 HTML 的网页

我们已经创建可以查看web目录下/html/markdown.html

> 2、使用迅雷下载显相关的css和js代码库文件

有三个文件jquery.js、markdown-it.min.js、skeleton.css。
**注意** 其中的jquery.js可以是本地已经下载好的jquery.
    
地址为:
* http://cdn.bootcss.com/jquery/3.0.0-alpha1/jquery.js
* http://cdn.bootcss.com/markdown-it/4.4.0/markdown-it.min.js
* http://cdn.bootcss.com/skeleton/2.0.4/skeleton.css

下载完成后拷贝放到web的/plug/markdown目录下

> 3、web应用部署访问

访问测试
http://localhost:8080/html/markdown.html?md=/docs/D3关系图说明文档.md

注意：这是web目录下的/docs/D3关系图说明文档.md转换为HTMl访问查看

> 图为：最终测试展示图

![](/images/markdown/test.jpg)











