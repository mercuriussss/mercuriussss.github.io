---
title: SpringBoot项目中关于maven依赖冲突的BUG处理(搞了一天的BUG)
date: 2019-05-12 01:04:11
tags: [springboot,bug,maven]
---
### 第一个BUG

首先是原初之罪的BUG，出现了找不到Bean的情况

![d2e1a8ec8c959c5b8079cdd030f938f2.jpeg](SpringBoot-1/1.jpg)

![bb5fc51a0ca7e3047a08f22768885e58.png](SpringBoot-1/2.png)

中间是怎么解决的我忘记了，还引出一个新建的maven module没有配置`Sources Root`、`Test Sources Root`等相应文件夹，后面这个通过`Project Structure`的Modules设置和Module自身的`.imi`文件设置就能解决

### 第二个BUG

解决了这些后，又TM引出了一个BUG

![32f7aeaf6df06b606f7d856afb5c3a43.jpeg](SpringBoot-1/3.jpg)


无论都导入不了`tk.mybatis.mapper.common.Mapper`包

![78c2033d6d8888c7b3449d8e79deccce.png](SpringBoot-1/4.png)

搞了半天都没处理好的我，毅然决然地使用了暴力破除法，直接重建一个项目，再将这个项目的东西搬过去，然后就好了……

![6804a1bfd13a933e956ce57d118d655a.jpeg](SpringBoot-1/5.jpg)

本以为这样终于结束了，因为到这里已经花掉我半天时间了

——但！是！

![b20c8506f2ee9d6a261b6e4604f94bf4.jpeg](SpringBoot-1/6.jpg)

### 第三个BUG

这里又双叒叕冒出来一个BUG了

![9e90f455f45ebbed3c8139b41f1f110a.jpeg](SpringBoot-1/7.jpg)

![bcd9aced33b70b25aba20f0aec0e4d12.png](SpringBoot-1/8.png)

![0df1b442e2f7e6c593e4d1ac05bc4ad3.png](SpringBoot-1/9.png)

![77dcd3abf10610a79d5833f9d84ac675.png](SpringBoot-1/10.png)

查看了下出错的源代码，很明显的`NoSuchMethodError`，找不到对应方法，源代码也是干脆利落地给出了红色标注我当场就懵了，总不会是源代码自己出错吧，没理由啊，肯定是自己太菜了，哪里弄错了才对

![a4f69231210ab8cb51b1884d37802657.png](SpringBoot-1/11.png)


于是我又继续埋头苦干，边自己捣鼓，边去群里询问
期间从某个学习群的群友那里得到了一个可能是导错包了的原因，然后我想错了，以为是项目对于maven仓库包的引用导入有问题，需要更新下什么的，就把整个maven本地仓库删掉再重加载，再把项目的`Libraries`引用全删了再重新用`mvn compile`指令加载回来

然后不出意外的——问题并没有得到解决
（理所当然的，因为问题根本就不是出在那里）

![b5d9abefeaed36b137494ee2e77ffc7f.jpeg](SpringBoot-1/12.jpg)

最后发现是`pom.xml`配置文件里面的依赖冲突导致的，将其那段多余的冲突依赖删掉后就没事了

![c26f422bafc783c442ad6431774e7bf0.png](SpringBoot-1/13.png)

![18a54ef3a379e6292ede09ed488574a6.png](SpringBoot-1/14.png)

### 处理依赖冲突的maven helper插件

最后有个群友推荐了IDEA的一个专门处理maven依赖冲突的插件，`maven helper`，下载好了后重启，即可使用

![e39647badb576da80b60668ba938554d.png](SpringBoot-1/15.png)

重启后进入`pom.xml`配置文件内，可看到以下内容

![bd4b7cbf67899e520830843266f12890.png](SpringBoot-1/16.png)

使用时，右键可以将用不到的版本`Exclude`掉，不过要解决问题大多没那么简单直接`Exclude`就能完事的，相关教程什么的可以去网上找，这里就不多说了

![4173b9cb6ffa4c081718e22073b9ac53.png](SpringBoot-1/17.png)

