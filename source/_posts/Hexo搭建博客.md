---
title: Hexo搭建博客
date: 2017-01-06 21:48:17
tags:
---
## Hexo 搭建个人博客
--------
Hexo是基于nodejs的工具，需要先安装nodejs

1. windows下自己接下载nodejs的安装包，安装即可。
2. nodejs自带npm工具。
3. 使用npm安装Hexo。
{% codeblock install-hexo lang:sh %}
npm install -g hexo-cli
{% endcodeblock %}

4. 初始化博客的目录
{% codeblock %}
hexo init [directory]
{% endcodeblock %}

5. 添加文章
{% codeblock %}
hexo new name
{% endcodeblock %}

6. 部署
{% codeblock %}
hexo deploy --generate
{% endcodeblock %}

写在最后的问题：
* 绑定域名，使用的是二级域名
    1. 购买域名，略过
    2. 添加 CNAME 文件到github的项目，内容写自己的域名即可
    3. 在域名解析中添加一条cmane的记录，指向 **[name].github.io**, name替换为自己的名字，即可