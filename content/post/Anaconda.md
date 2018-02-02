+++
title = "Anaconda:一个可用于科学计算的Python发行版"
description = "Anaconda 是一个可用于科学计算的Python 发行版，支持Linux、Mac、Windows系统，内置了常用的科学计算包。"
tags = [
    "Python",
    "anaconda",
]
date = "2018-02-02"
categories = [
    "Development",
    "language",
]
+++


Anaconda 是一个可用于科学计算的Python 发行版，支持Linux、Mac、Windows系统，内置了常用的科学计算包。

<!--more-->

## Anaconda是什么?

Anaconda在英文中是“蟒蛇”，麻辣鸡（Nicki Minaj妮琪·米娜）有首歌就叫《Anaconda》，表示像蟒蛇一样性感妖娆的身体。

![](/images/python/anaconda.jpg)

-   1）Anaconda 附带了一大批常用数据科学包，它附带了 conda、Python 和 150 多个科学包及其依赖项。因此你可以立即开始处理数据。
-   2）管理包Anaconda 是在 conda（一个包管理器和环境管理器）上发展出来的。在数据分析中，你会用到很多第三方的包，而conda（包管理器）可以很好的帮助你在计算机上安装和管理这些包，包括安装、卸载和更新包。
-   3）管理环境为什么需要管理环境呢？比如你在A项目中用了 Python 2，而新的项目B老大要求使用Python 3，而同时安装两个Python版本可能会造成许多混乱和错误。这时候 conda就可以帮助你为不同的项目建立不同的运行环境。

##  Anaconda常用命令

-   conda --version
-   conda help
-   conda info
-   conda list
-   conda upgrade --all (初次安装下的软件包版本一般都比较老旧，因此提前更新可以避免未来不必要的问题。)

##  如何管理包？

### （1）安装包

```conda install pandas numpy```

### (2)卸载包

```conda remove panda numpy ```

### (3)更新包

```conda update pandas```

##  如何管理环境？

conda 可以为你不同的项目建立不同的运行环境。

### （1）安装nb_conda用于notebook自动关联nb_conda的环境

```conda install nb_conda```

### （2）创建环境

``` conda create -n py3 pandas ``` 创建名为py3的环境，并在其中安装pandas

```conda create -n py2 python=2 ```

```conda create -n py3 python=3 ```

```conda create -n py python=3.6 ```

### (3)进入环境

```source activate py3 ```

### (4)离开环境

```conda deactivate```

### (5)列出环境

```conda list env```

### (6)删除环境

```conda env remove -n py2```

##  用pip安装包

conda安装不了pandas_datareader,改用pip安装：

```pip install pandas_datareader```