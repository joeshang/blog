---
layout: 
title: CNAME 有什么用？
date: 2020-06-14 17:01:44
categories: Network
tags:
---

之前一直好奇，公司用的 CDN 是公司域名，是如何转到阿里云或者腾讯云的？后来翻看了 DNS 的一些知识，发现和 CNAME 有关：

CNAME 是 Canonical Name  的缩写，也成为别名指向。

DNS 中 CNAME 记录和 A 记录的区别在于，A 记录是把一个域名解析到一个 IP 地址（Address，这也是 A 记录名字的原因），CNAME 记录是把一个域名解析到另一个域名，相当于加了一个中间层。

通过 dig 能看到 DNS 域名解析的过程，具体见：https://www.ruanyifeng.com/blog/2016/06/dns.html

CNAME 有什么用？有句话是”在软件开发中，没有什么是加一个中间层搞不定的，如果不行，就再加一层“，哈哈，开个玩笑。CNAME 加中间层的好处是：
* 多个域名都指向同一个别名，当 IP 变化时，只需要更新该别名的 IP 地址（A 记录），其他域名不需要改变
* 有的域名不属于自己，例如 CDN 服务，服务商提供的就是一个 CNAME，将自己的 CDN 域名绑定到 CNAME 上，CDN 服务提供商就可以根据地区、负载均衡、错误转移等情况，动态改别名的 A 记录，不影响自己 CDN 到 CNAME 的映射。


