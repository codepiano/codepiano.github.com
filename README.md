# 个人博客

[codepiano](http://codepiano.github.io)

个人博客，转载请注明出处，保留所有权利。

## 使用的工具

1. 图标库         [Font-Awesome](http://fortawesome.github.io/Font-Awesome)

1. 静态页面服务   [Github pages](http://pages.github.com)

1. 博客生成工具   [Jekyll](https://github.com/mojombo/jekyll)

1. 博客生成工具   [Jekyll-Bootstrap](http://jekyllbootstrap.com/)

1. 前端工具库     [JQuery](http://jquery.com/)

1. 表格控件       [DataTables](http://www.datatables.net/)

1. 前端框架       [Twitter Bootstrap](http://twitter.github.io/bootstrap)

1. 前端排版样式表 [Typo.css](http://typo.sofish.de)

1. 开发工具       [Vim](http://www.vim.org/)

## 使用方法

### 通过clean分支（建议）

提供了一个clean分支，直接检出clean分支修改设置，然后merge回master分支即可

所有的个性化变量都被替换为字符串"site"

### 通过fork

1. fork我的博客，修改fork后的项目名称为：你的github ID.github.com
1. 修改fork后的远程仓库地址：git remote set-url origin 替换为你的仓库地址
1. 参见清理文件并修改配置

### 通过clone

1. git clone https://github.com/codepiano/codepiano.github.com.git 替换为你的目录名
1. git remote set-url origin 替换为你的仓库地址
1. 参见清理文件并修改配置

### 清理文件并修改配置

清理文件步骤

1. 删除\_post目录下所有文件
1. 删除pages目录
1. 删除CNAME文件，如果你需要自定义域名，可以修改CNAME文件
1. 修改config.yml中的设置，需要自定义的地方已经加了注释，建议把那个文件看一遍，对设置有个大概的了解
1. 我使用我的id作为了一些设置的属性名、文件名、目录名，如果你想修改，最好使用替换工具，把所有文件中的"codepiano"，替换为你想要的名字 然后重命名以我的id为名的所有文件和目录
1. config.yml中的comments和analytics必须修改，配置上你自己的账号，如果不想配置，请置provider为false
1. 出现问题，建议自己google，有很多详细的教程，或者直接参考官方文档 [jekyll](http://jekyllrb.com)
1. 有好的建议或者要求，欢迎提issue或者发邮件交流

## 注意

### 归档页面链接图标

归档页面文章前的符号是font-awesome的图表名称，请在post中的yaml指定icon属性

比如想展示class名字为icon-github的图标，指定icon属性值为github即可,具体请参考post中的写法

如果使用rake文件生成post，post默认的图标是file-alt

### 导航栏

为了自定义导航栏子栏目的顺序，重写了这部分的逻辑，具体在文件\_include/codepiano/navigation\_list中

为了在后台实现高亮当前的导航页，用一种不太好的方式实现了这个功能，建议使用js在页面加载后进行设置

用我自己使用的导航栏来作为示例，我的导航栏有四个，对应文件在根目录下，均为html文件

1. 文章 /posts.html

1. 时间线 /timeline.html

1. 目录 /categories.html

1. 关于 /about.html

首先color\_hack这个变量存放导航栏的文件名， `{% assign color_hack = 'posts timeline categories about' %}`

比如点击导航栏的‘文章’链接，page.url变量的值是/posts.html，将这个字符串去除‘/’，去除‘.html’，得到字符串posts，存入current\_nav变量

然后将color\_hack中的current\_nav替换为'active'，即将posts替换为active，得到color\_hack的指为'active timeline categories about'

然后将数组分割，得到一个数组[active,timeline,categories,about]，赋值给color\_hack

然后按顺序指定导航栏的html内容，顺序应与color\_hack初始值中的顺序相对应，class属性的指即从对应的color\_hack中按顺序获取

1. `<li class="{{color_hack[0]}}"><a href="/posts.html">文章</a></li>`

1. `<li class="{{color_hack[1]}}"><a href="/timeline.html">归档</a></li>`

1. `<li class="{{color_hack[2]}}"><a href="/categories.html">目录</a></li>`

1. `<li class="{{color_hack[3]}}"><a href="/about.html">关于</a></li>`

这样当前访问的导航的class会被指定为active，其他导航的class会被指定为各自的文件名
