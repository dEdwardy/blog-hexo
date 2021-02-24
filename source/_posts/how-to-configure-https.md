---
title: Https配置实战(阿里云 + nginx)
date: 2021-02-22 22:10:28
tags: https
thumbnail: /img/20210222/bg.jpg
---
## 为什么需要Https
## Https的配置
### 证书申请

申请证书的方案很多，可以从[Let's Encrypt](https://letsencrypt.org/)申请，由于我的云服务器是从阿里云上购买的，所以直接在阿里云上申请免费证书（自 2021 年起，免费证书申请切换到证书资源包下每个实名个人/企业主体在一个自然年内可以一次性领取 20 张免费证书）
{% fb_img /img/20210222/1-1.png 证书申请-1 %}

{% fb_img /img/20210222/1-2.png 证书申请-2 %}

填写域名 联系人等相关信息
{% fb_img /img/20210222/1-3.png 证书申请-3 %}

{% fb_img /img/20210222/1-4.png 证书申请-4 %}

此时，将得到的 dns 记录信息录入阿里云 的 DNS解析中

### 上传证书

待证书签发成功后，在阿里云的证书资源包选择对应的 web 服务器并点击下载证书，并将下载好的文件上传到 nginx conf 目录下的 cert 目录中(若此目录没有则新建 cert 目录)

### 配置 nginx conf

配置 conf 目录下的 nginx.conf 文件
{% fb_img /img/20210222/2-1.png Nginx配置-1 %}  

{% fb_img /img/20210222/2-2.png Nginx配置-2 %}  
然后再加上重定向

```
return 301 https://$server_name$request_uri;
```

### 配置端口

接下来配置一下云服务器的安全组规则 开放 443 端口
最后打开浏览器输入域名验证一下效果:
 
{% fb_img /img/20210222/3.png https配置效果图 %}  

## 总结
https的配置分为以下几个步骤
1. 申请证书 
2. 上传证书至Web服务器
3. Web服务器修改配置
4. 开放端口，验证