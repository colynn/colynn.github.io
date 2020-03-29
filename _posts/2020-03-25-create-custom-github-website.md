---
layout: post
title: 如何创建自己的github站点
subtitle: 站点模板选择/自动初始化gittalk评论组件
tags: [blog, Gittalk]
comments: true
---

## 站点模板

你可以根据[jekyllr themes](https://jekyllrb.com/docs/themes/)选择自己喜欢的模板， 或是创建自己的主题并可以让更多人使用它。

## 评论组件
大多模板已经支持一些评论组件，`disqus`/`staticman`等，但是考虑到国内的网络环境，就接入了[Gittalk](https://github.com/gittalk)这个评论组件。

可以参看[Gittalk文档](https://github.com/gitalk/gitalk#Install), 由于作者使用的是jekyll类型的主题，为方便标准化做一些说明。

### 1. 定义配置参数至`_config.yml`

```

# To use Gittalk comments  https://github.com/gitalk/gitalk#Usage
gittalk:
  # Note: A GitHub Application is needed for authorization, if you don't have one, going to https://github.com/settings/applications/new register a new one.
  # You must specify the website domain url in the Authorization callback URL field.
  clientID: 7196948b9a586a342072 # GitHub Application Client ID
  clientSecret: 1acf6a13bc1ce7b8d8e3554789781ba2aaa4575c  # GitHub Application Client Secret,
  repository: blog-comments  # Storage gittalk's repository
  owner: colynn  # GitHub repo owner, 一般情况就是你的github帐号
  admin: colynn  # GitHub repo owner and collaborators, only these guys can initialize github issues eg. 'colynn,daattali' 如果你不需要让更多人有权限初始化gittalk评论时，这里也是只填写自己的github帐号就好。

```

### 2. 定义gittalk子页面至`_includes/gittalk-comment.html`

```
{% if site.gittalk.clientID and site.gittalk.clientSecret %}
<div id="gitalk-container"></div>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<script >
  const gitalk = new Gitalk({
    clientID: '{{ site.gittalk.clientID }}',
    clientSecret: "{{ site.gittalk.clientSecret }}",
    repo: "{{ site.gittalk.repository }}",
    owner: "{{ site.gittalk.owner }}",
    admin: ["{{ site.gittalk.admin | split: ','  | join: '","'}}"],
    id: "{{ page.url }}",    // Ensure uniqueness and length less than 50 
    distractionFreeMode: false,  // Facebook-like distraction free mode
    perPage: 100
});

gitalk.render('gitalk-container')
</script>

{% endif %}

```

### 3. 引入至相应的页面`_layouts/page.html、_layouts/post.html`

```
{% include gittalk-comment.html %}
```

### 4. 保存变更提交，重新部署
在你已经引入gittalk的页面，首页访问时会显示如下页面：

你只需要登录帐号（就是定义在gittalk.admin下用户），页面刷新下初始化，之后其他人阅读者就可以评论了。


## 评论组件2
如果你觉得 Gitalk 都只能手动初始化所有文章的评论或者一个一个点开界面，如果觉得这件事情非常麻烦，可以手动抓了一下Gitalk 在初始化评论时发出的网络请求写一个用于自动化初始评论的脚本。

其实也可以使用另外一种组件-[utterances](https://github.com/utterance/utterances), 目前utterances不支持引用回复，其他的


功能特性 | Gitalk |  Utterances  
-|-|-
新用户评论入口 | 评论页面的顶部 | 评论页面的底部 |
引用回复 | 支持 | 目前暂不支持[#260](https://github.com/utterance/utterances/issues/260)|
对于别人的留言添加动作 | 不支持 | 支持 |
页面引入后初始化操作 | 人为刷新/或自备脚本 | 不需要初始化，首评时自动初始化|





