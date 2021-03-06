---
title: Charles 抓包（http/https）
categories: ["工具"]
tags: ["tools"]
date: 2019-02-13
lastmod: 2019-02-13
---


### 运行环境
* Mac os 10.13.6
* Ios 12.1.2(16C101)

### 下载安装
* [官网下载](https://www.charlesproxy.com/download/)
* _注_: 破解版麻请自行搜索下载

### 配置
1. Proxy -> Proxy Settings (根据需要调整配置，保存)；

__注__: 如果仅需要抓取http包，请略过后续步骤。

2. Proxy -> SSL Proxying Settings

![image](https://user-images.githubusercontent.com/5203608/89494735-67614080-d7e9-11ea-877e-a642bec7ed66.png)


3. 电脑端安装证书 Help -> SSL Proxying -> Install Charles Root Certificate，并将证书标记为始终信任

![image](https://user-images.githubusercontent.com/5203608/89494759-75af5c80-d7e9-11ea-94c2-d711e086ecd5.png)

4. 手机端安装证书, 用Safari浏览器打开： chls.pro/ssl ；提示下载， 然后安装
__注__: IOS 10.0之后需要在 设置 -> 通用 -> 关于本机 -> 证书信息设置， 信任该证书就可以了。

5. 获取代理信息：
通过如下方式获取代理信息： Help -> SSL Proxying -> Install Charles Root Certificate on a Mobile Device or Remote Browser，
显示如下图 代理信息为： 172.16.17.35:8888

![image](https://user-images.githubusercontent.com/5203608/89494775-7ba53d80-d7e9-11ea-86ce-254a686e5a3e.png)

6. 手机端配置代理：
设置 -> 无线局域网 -> 网络id -> 配置代理 -> 手动, 如下图

![image](https://user-images.githubusercontent.com/5203608/89494785-8069f180-d7e9-11ea-8d1a-7e2d8a0d98e7.png)






