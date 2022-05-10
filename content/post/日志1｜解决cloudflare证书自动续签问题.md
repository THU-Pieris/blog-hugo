---
title: "日志1｜解决cloudflare证书自动续签问题"
date: 2022-05-10T16:10:33+08:00
image: 
math: 
license: 
comments: true
slug: 5df638b0
categories:
  - tech
tags:
  - mastodon
description: 都是Automatic HTTPS Rewrites干的好事
---

> 本文在[Craft](https://www.craft.do)上撰写并更新。可访问以下链接：
>  
> - 原文: <https://www.craft.do/s/WhfmCHQlkCpCFI>

在网站开启了cloudflare之后，证书续签可能会出现以下问题：

```Bash
~$ sudo certbot renew --dry-run
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/mstdn.pieris05.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for mstdn.pieris05.com

Certbot failed to authenticate some domains (authenticator: nginx). The Certificate Authority reported these problems:
  Domain: mstdn.pieris05.com
  Type:   unauthorized
  Detail: <an IPv6 address>: Invalid response from https://mstdn.pieris05.com/.well-known/acme-challenge/XXXXXXXXXXXXXXXXXXXXXXXX: 404

Hint: The Certificate Authority failed to verify the temporary nginx configuration changes made by Certbot. Ensure the listed domains point to this nginx server and that it is accessible from the internet.

Failed to renew certificate mstdn.pieris05.com with error: Some challenges have failed.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
All simulated renewals failed. The following certificates could not be renewed:
  /etc/letsencrypt/live/mstdn.pieris05.com/fullchain.pem (failure)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1 renew failure(s), 0 parse failure(s)
Ask for help or search for solutions at https://community.letsencrypt.org. See the logfile /var/log/letsencrypt/letsencrypt.log or re-run Certbot with -v for more details.
```

搜索后发现，问题出在cloudflare上设置了`Automatic HTTPS Rewrites` 。「在尝试通过http方式下载challenge文件的时候，被直接重定向到了https，因此产生了问题。由于源服务器仅在80端口配置了acme-challenge，所以当从https访问的时候就得到了404错误。」[^1]所以，解决思路有如下几个：

- 关闭Automatic HTTPS Rewrites
- 关闭cloudflare
- 证书到期时暂时关闭cloudflare，手动续签后再打开
- 添加一条Page Rules

显然，最后一条解决方法才是比较推荐的。cloudflare免费版可以添加最多3条Page Rules，用以对匹配到的URL进行规则修正。

![aa737566df5af6a04341026523af6ca43cd01c54](https://cdn.jsdelivr.net/gh/THU-Pieris/Image@main/img/aa737566df5af6a04341026523af6ca43cd01c54.1e2znvpo0mzk.png)

按照上图配置好后，再次尝试续签证书，成功！

```Bash
~$ sudo certbot renew --dry-run
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/mstdn.pieris05.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for mstdn.pieris05.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded: 
  /etc/letsencrypt/live/mstdn.pieris05.com/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

[^1]: [https://archive.ph/aIhq4](https://archive.ph/aIhq4)
