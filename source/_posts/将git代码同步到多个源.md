---
title: 将git代码同步到多个源
date: 2023-03-04 21:19:14
tags:
---

应用场景，将github的开源代码同步到国内gitee源，需要同步所有分支以及提交。
使用git mirror方案。

### 将github项目更新到本地
命令如下，一定要加上--mirror参数
```
git clone --mirror https://github.com/xxx.git
```

### 更新并推送到gitee
push 时一定要带上--mirror
```
git fetch -p origin #更新到最新
git push --mirror https://gitee.com/xxx.git
```

这个脚本加入到定时脚本，后续就能定时同步代码了。 