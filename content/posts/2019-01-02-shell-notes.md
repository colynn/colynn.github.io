---
title: Shell命令备注手册
categories: ["教程"]
tags: ["shell", "运维"]
date: 2019-01-01
lastmod: 2021-01-26
---

## Sed

### 直接编辑文件

```sh
sed -i 's/abc/xxx/g' file
sed -i '/$xxx.*$/d' file
```
__注__: 仅仅工作在linux类型的主机上