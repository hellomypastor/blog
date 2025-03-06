title: Hello World
date: 2015-05-07 21:28:51
type: "tags"
tags:
- 心路札记
---
一直想搭建一个属于自己的博客，本来想基于`Python`搭建的，但无奈云服务器太贵，所以只能放弃，后来看到了这篇文章[通过Hexo在Github上搭建博客教程](http://andrewliu.in/2014/11/21/%E9%80%9A%E8%BF%87Hexo%E5%9C%A8Github%E4%B8%8A%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2%E6%95%99%E7%A8%8B/)，顿时眼前一亮，这个博客终于搭建起来了。
折腾了两天多，在github page上看到自己的博客加载下来时，突然有种错综复杂的恍惚感。是的，它不是qq空间，不是新浪博客，不是豆瓣小站，也不是贴吧。它更像是属于自己的一块小小的领地，因而我满足于这种归属感。我愿在上面安静劳作。
一个农民，通过自身努力终于分到了一块地，不再需要在地主的土地上创造流量价值时，于是翻身作主的他可以宣告说：Hello World。当然这个农民确切来说是个码农。
感谢[Hexo](http://hexo.io/)框架，感谢[Litten](http://litten.github.io/)提供的[Yilia](https://github.com/litten/hexo-theme-yilia)主题，我很喜欢这种色调。
<!--more-->

## 附上搭建过程中遇到的问题

### 环境搭建

``` 
#第一次使用执行前需执行 npm install
hexo generate  #自动根据当前目录下的文件，生成静态网页
```

### 部署到Github(`_config.yml`)

``` 
#执行
hexo deploy
#出现如下错误
error deployer not found:github
#要将_config.yml中github改成git
deploy:
  type: git    #部署类型, 本文使用Git
#并在执行hexo deploy前先执行
npm install hexo-deployer-git --save
```

More info: [搭建 hexo，在执行 hexo deploy 后,出现 error deployer not found:github 的错误](http://www.v2ex.com/t/175940)

### 绑定域名(新建`CNAME`文件)

```
#在自己的域名管理界面添加CNAME域名解析
#在public目录下新建CNAME文件
hellomypastor.net
```

