- Date: 2022-01-17
- Time:  08:40




## **加速地址一览**

-   **[fastgit.org](https://link.zhihu.com/?target=https%3A//hub.fastgit.org/)：**[https://doc.fastgit.org/](https://link.zhihu.com/?target=https%3A//doc.fastgit.org/)
-   **[http://cnpmjs.org](https://link.zhihu.com/?target=http%3A//cnpmjs.org)：**[https://github.com.cnpmjs.org/](https://link.zhihu.com/?target=https%3A//github.com.cnpmjs.org/)
-   **[http://gitclone.com](https://link.zhihu.com/?target=http%3A//gitclone.com)：**[https://gitclone.com/](https://link.zhihu.com/?target=https%3A//gitclone.com/)
-   **gitee：**[https://gitee.com/mirrors](https://link.zhihu.com/?target=https%3A//gitee.com/mirrors)
-   GitHub 文件加速：[https://gh.api.99988866.xyz/](https://link.zhihu.com/?target=https%3A//gh.api.99988866.xyz/)
-   Github 仓库加速：[https://github.zhlh6.cn/](https://link.zhihu.com/?target=https%3A//github.zhlh6.cn/)
-   Github 仓库加速：[http://toolwa.com/github/](https://link.zhihu.com/?target=http%3A//toolwa.com/github/)

## **加速** **`clone`**

```text
# 方法一：手动替换地址 
#原地址 
$ git clone https://github.com/kubernetes/kubernetes.git 
#改为 
$ git clone https://github.com.cnpmjs.org/kubernetes/kubernetes.git 
#或者 
$ git clone https://hub.fastgit.org/kubernetes/kubernetes.git 
#或者 
$ git clone https://gitclone.com/github.com/kubernetes/kubernetes.git 
 
# 方法二：配置git自动替换 
$ git config --global url."https://hub.fastgit.org".insteadOf https://github.com 
# 测试 
$ git clone https://github.com/kubernetes/kubernetes.git 
# 查看git配置信息 
$ git config --global --list 
# 取消设置 
$ git config --global --unset url.https://github.com/.insteadof 
```

## **加速** **`release`**

```text
# 原地址 
wget https://github.com/goharbor/harbor/releases/download/v2.0.2/harbor-offline-installer-v2.0.2.tgz 
# 加速下载方法一 
wget https://download.fastgit.org/goharbor/harbor/releases/download/v2.0.2/harbor-offline-installer-v2.0.2.tgz 
# 加速下载方法二 
wget https://hub.fastgit.org/goharbor/harbor/releases/download/v2.0.2/harbor-offline-installer-v2.0.2.tgz 
```

  

## **加速** **`raw`**

```text
# 原地址 
$ wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/README.md 
# 加速下载方法一 
$ wget https://raw.staticdn.net/kubernetes/kubernetes/master/README.md 
# 加速下载方法二 
$ wget https://raw.fastgit.org/kubernetes/kubernetes/master/README.md
```





[参考自：](https://zhuanlan.zhihu.com/p/420873495)