---
title: 此博客的'构造方法'
date: 2016-11-04 18:22:18
categories: node
tags:
 - github.page
 - hexo
 - git
 - node
---
*其实这就是熟悉熟悉怎么用git，不建议一个仓库弄两个分支...*

HEXO初始化
-------

暂设当前目录 iamtb，且github仓库A放hexo dynamic文件，仓库B放hexo static文件

    npm install -g hexo

    npm install hexo-deployer-git --save

    hexo init iamtb

 在iamtb文件夹下

    hexo s


来个NB的NEXT主题
-----------

在iamtb目录下载主题NEXT，

<!-- more -->

    git clone https://github.com/iissnan/hexo-theme-next themes/next

修改基本主题配置，包括语言，page等，确保本地预览无误。

[重点来了！！！]
-------

[重点来了！]必须**删除下载主题内的.git目录**，否则主题无法上传到A库，因为你自己下载的主题文件有.git控制文件夹。
另外一定**修改.gitignore** ,把其中的`node_modules`和`db.json`，不要ignore，不要ignore，不要ignore。

 意思就是你的`.ignore`文件是这样：

    .DS_Store
    Thumbs.db
    *.log
    public/
    .deploy*/

推到A仓库
-----

在github新建[A仓库][1]，把目前的这些文件都推到A库。参考以下命令

      503  git status
      504  git init
      505  git status
      506  git add *
      507  git commit -m 'init hexo and next theme'
      508  git status
      509  git remote
      510  git remote add origin https://github.com/tuber/tbhexo.git
      511  git branch
      512  git push
      513  git push origin master

建立yourname.github.io
--------------------

github建立[仓库B][2],建完后可一进入仓库setting进行初始化（Launch automatic page generator），保证访问`yourname.github.io`正常访问。



本地config配置
----------

 目的就是执行`hexo d`的时候推到B仓库

    deploy:
      type: git
      repo: https://github.com/tuber/tuber.github.io.git
      branch: master
      message: just static




本地发布测试文章
--------

 如果网络慢可以在主题设置中禁用google字体

    hexo new ‘001’

    hexo new ‘002’

    hexo new ‘003’

    hexo g

    git commit -am'add 3 post'

    git push origin master #把本地master推到远程origin，此处为A仓库

    hexo d #把生成静态文件推到B仓库

完成80%了，如果换电脑了呢？
-------

 那我们由于之前都做好了准备，我们只需要在你需要的地方

    git clone https://github.com/tuber/tbhexo.git，

 这样你目录下就会新增了一个`tbhexo`文件夹

进入thhexo目录，**不用init 不用init 不用init**

    npm install hexo

    npm install hexo-deployer-git

然后还是先`hexo s` 启动一下，正常没问题

 然后从tbhexo文件夹下，（就相当于你另外一台电脑了）
随便创建几个文件：

    hexo new add tbhexo 004
    hexo new add tbhexo 005
    hexo new add tbhexo 006

可以参考以下命令



      504  git status
      505  git add *
      506  git commit -m'add 3 posts and tow helloword posts in tbhexo'
      507  git push origin master

最后一步
----

`hexo d`


完工。这样你在另外一台电脑上更新的文章也推到`yourname.github.io`上了。



如此循环
----

只需要  `git pull origin master`更新

`git push origin master` 上传即可。

大问题：
----

目测他有一个默认文章hello word ，理论应该在第一个文章，但是经过此番折腾他跑到第三文章了。
贴图：

![图片描述][3]


  [1]: https://github.com/tuber/tbhexo.git
  [2]: http://tuber.github.io
  [3]: /img/node/IAMTB.png
