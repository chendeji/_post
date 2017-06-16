---
title: 使用Github page配置个人博客
date: 2014-04-24 11:43:07
tags: [bolg, github]
categories: [github]
---

## 具体配置注意点

配置Github page 的很多博客都有很多资料了基本大同小异，这里推荐一个地址 : [使用Hexo搭建个人博客](http://opiece.me/2015/04/09/hexo-guide/)。

使用[Hexo](https://hexo.io/)上传你说写的博客时，特别要注意的一点就是hexo项目根目录下的_config.yml这个文件中的这一段：
```
deploy:
  type: git
  repository: git@github.com:用户名/用户名.github.io.git
  branch: master
```

上面的repository这里，如果较早版本的配置，不是这样的，所以最好确保你的[Node.js](https://nodejs.org/en/)版本是最新的，到官网上下载最新的版本，就直接按照这个格式，将自己的用户名输入，然后

### 上传最新文章
``` bash
$ hexo d
```

就能将你的博客文章上传到github。整个配置过程还是相对比较简单的。


