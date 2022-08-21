---
draft: false
date: 2022-03-22T07:58:57+08:00
title: 新的博客
description:
slug: new-blog-new-start
categories: 
    - Journal
tags: 
    - misc
---

> 好久没有写博客了，可能是懒惰也可能是生疏导致的，但是客观地来讲并不是一个很好的现象，因为知识是网状的需要总结需要成体系的。当然也不是所有的内容都适合被结构化，也有一些其他的有意思的但是值得被记录下来的东西。

我的个人博客从最早到现在也算是有很长的历史了。
部署方式从最早的 空间 到 独立主机 到 SAE 到 云 到 Github Pages 到 Netlify 到 最后又回到了Github Pages（其中有很多的原因，但是这篇文章不会进一步说明）；
博客系统从 Wordpress 到 Ghost 到 Echo 到 Hexo 到 Hugo 到 Hexo 然后目前又回到了 Hugo；
域名从 `www.d0zingcat.xyz` 到 `blog.d0zingcat.xyz` 到 `infloop.life` 到 `blog.d0zingcat.xyz` 到 `blog.d0zingcat.dev`。之所以最后选择用dev的域名原因有那么几个：
1. dev是Google Domains买的，一年12刀很便宜，谷歌爸爸赛高。
2. 可以自动签https证书，且可以无限添加跳转规则（对比之下Cloudflare有点过于小气了），跳转带子路径，比如 `https://d0zingcat.dev/` 会自动跳转到 `https://github.com/d0zingcat/` ，所以要访问我的个人项目比如dotfiles只需要访问 `https://d0zingcat.dev/dotfiles` 就会自动跳转了。
3. .dev 更复合我开发的身份。
4. 域名包括博客和内容，每次我重新调整的时候基本是一个全新的开始，是一个美好的祈愿，也是对自己的一种鼓励和鞭策。

如果关注想知道我怎么部署的，不如参考我的仓库吧：[d0zingcat.github.io][2] 。这个仓库实现了自动打包发布到 Github Pages 和 [Fleek][4] 上，主要还是根据官方的一些教程进行操作的，不是很难，如果没有特别的需要不会展开写怎么设置这些。另外，如果你很感兴趣的话，可以访问我部署在 IPFS 上的博客，和这个博客是镜像： [d0zingcat.eth.limo][3] 。部署在Fleek上有两个注意点： `Settings-Build Settings` 中设置`Build Command` 为 `git submodule update --recursive && hugo`，而 `Advanced Build Settings-Specify Docker Image` 则使用 `shawnoster/hugo-extended-for-fleek:99.0` 这个镜像，因为我的主题用了`Sass/SCSS` 需要`hugo-extended` 才能支持生成。

值得一提的是，部署在 fleek 上配置好对应的 ENS 域名之后，会同时获得三个域名： `xxx.on.fleek.co`, `xxx.eth.link`, `xxx.eth.limo`。例如V神的博客地址就是 [vitalik.eth.limo][5]。和V神一起做邻居的感觉真好。

这次重新开始我准备做出几个比较大的改变：
1. 少说废话
2. 少写教程
3. 面向有基础的读者，很多基础的操作和概念不再做细致的说明
4. 多写干货，把博客当成总结

希望这是一个新的开始，New Blog, New Start.

# 参考

[使用 GitHub Actions 自动部署 Hexo 博客][1]

[Override Archetypes][6]

[1]: https://printempw.github.io/use-github-actions-to-deploy-hexo-blog/
[2]: https://d0zingcat.dev/d0zingcat.github.io
[3]: https://d0zingcat.eth.limo
[4]: https://fleek.co/
[5]: https://vitalik.eth.limo
[6]: https://gohugobrasil.netlify.app/themes/customizing/
