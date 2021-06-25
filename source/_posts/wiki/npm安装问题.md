---
title: npm安装其他包报错
date: 2021-06-18
tags: [npm]
categories: npm
---

> 错误描述：当我使用` npm install wangeditor@4.6.3`安装这个包是报如下错误，网上的各种诸如重新设置代理都不见效
>
> ```shell
> npm ERR! code ECONNRESET
> npm ERR! errno ECONNRESET
> npm ERR! network request to http://registry.npmjs.org/wangeditor failed, reason: socket hang up
> npm ERR! network This is a problem related to network connectivity.
> npm ERR! network In most cases you are behind a proxy or have bad network settings.
> npm ERR! network 
> npm ERR! network If you are behind a proxy, please make sure that the
> npm ERR! network 'proxy' config is set properly.  See: 'npm help config'
> npm ERR! A complete log of this run can be found in:
> npm ERR!     /Users/mr.stark/.npm/_logs/2021-06-18T08_44_30_596Z-debug.log
> ```

解决办法

正当我被折磨的死去活来想要放弃时，突然想到我之前用` brew`安装软件时遇到过类似的问题，原因是我用了` ClashX`这个软件，它启动时会生成一些环境变量，我们复制一下终端代理命令看看

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/npm-error.png">

粘贴一下如下所示：

```shell
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891
```

接下来我们依次删除这三个环境变量即可

```shell
unset https_proxy
unset http_proxy
unset all_proxy
```

接下来我们就可以开心的使用npm安装其他的包来使用啦！

```shell
+ wangeditor@4.6.3
added 3 packages from 1 contributor, removed 16 packages, updated 1 package and audited 1327 packages in 21.52s

89 packages are looking for funding
  run `npm fund` for details
```



