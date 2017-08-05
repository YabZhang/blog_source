---
title: buildout构建工具
date: 2017-07-25 08:40:10
tags:
 - buildout
 - automation
---

### prologue

{% blockquote buildout doc, http://docs.buildout.org %}

Buildout is a tool for automating software assembly.

* Run build tools to build software.
* Apply software and templates to generate configuration files and scripts.
* Application to all software phases, from development to production deployment.
* Based on core priciples:
  1. Repeatability
  2. Componentization
  3. Automation

{% endblockquote %}

buildout的是一款自动化构建工具。
由`Zope`团队开发维护。包名为`zc.buildout`。

`buildout`可以为应用构建独立的依赖环境。类似于`virtualenv`，但二者还有不同。
粗略地讲，`buildout`支持的功能更多更便于自动化而且具体定位有所不同。

### use case

#### install buildout

* 首先是安装buildout

`pip install zc.buildout`

使用`pip`安装`zc.buildout`包。

* 为项目配置buildout

```
cd project
buildout init
```

`buildout init`在项目目录中初始化。初始化后，目录下会多出一些目录和文件。
其中`buildout.cfg`是默认的`buildout`配置文件。具体的配置留待下文介绍。

`buildout init`会采用当前默认的python环境。也可以在隔离环境（如:`virtualenv`）中运行。
而且`buildout init`初始化并不会提供`bootstrap.py`文件。
这个文件由官方提供，为了便于自动化需要下载这个文件，并加到版本控制中。[传送门](https://bootstrap.pypa.io/bootstrap-buildout.py)

基本的安装工作到这里就完成了。下面来看看如何使用`buildout`来构建一个环境。

* 使用buildout构建应用环境

假定我们需要在一台新的机器上构建`buildout`应用环境。
我们只需要进入项目目录，运行启动`bootstrap.py`脚本即可。
`bootstrap.py`会自行完成`buildout`和依赖的安装，并按照`buildout.cfg`配置内容完成构建。

```
cd project
python bootstrap.py
```

前文提到`buildout`可以在系统环境或隔离环境(如:`virtualenv`)构建具体的应用环境。
官方推荐在沙盒环境中构建:

```
virtualenv -p python3 project
cd project
bin/python bootstrap.py
```

目录结构如下：

```
project/
   bootstrap.py  †
   buildout.cfg  †
   .installed.cfg
   parts/
   develop-eggs/
   bin/
       buildout
       mypython
   eggs/
   downloads/
```

只有`bootstrap.py`和`buildout.cfg`需要加入到代码库。

#### buildout.cfg配置

* 简单的demo:

{% gist 5adfe037c44555d4c2b4cebfe55eb2a9 %}

`[buildout]`这个块是必需的，可以在此指定起用的配置块信息；
`parts`中指定起用的块名字，数目任意，一行一个；
`develop`用于指定创建`egg`包的目录，指定目录必须包含一个`setup.py`文件用于打包。

`[event]`是自定义块；如`[event]`，`[server]`...
`recipe`对自定义块是必需的，可以是分发好的包，也可以自行制作。
最常用的就是`zc.recipe.egg`，用于按照块定义打包(egg)。
包含了完成块定义逻辑代码，如安装依赖、生成脚本等。
`eggs`指定了需要加载的egg包以及相关的依赖。
`interpreter`会创建一个包含eggs和依赖的执行环境，在`bin`目录下。

更多选项参考文档。

* 配置实例：

{% gist d172d3f67336ba1891dce2646cf5dfdd %}

`[buildout]`起用了`project`块，并在当前境目录打包。
`[project]`用于创建一个'project'的包，并提供一个执行环境。
`extra-paths`选项的值，为一个或多个路径，会被添加到环境解释器的搜索路径中。
`{buildout:directory}`值为当前构建目录。
`[versions]`部分指定依赖的版本要求。

`develop`指定的打包目录中必须包含`setup.py`文件用于打包。
`setup.py`文件包含了所有打包必须的信息，如版本，依赖，和入口等。
`install_requires`中指定了依赖，如果缺失会使用`pip`安装。这里我采用了在`buildout.cfg`中进行包的版本管理。
`entry_points`中指明了包的入口，`buildout`会创建一个此入口的启动脚本在`bin`目录下。

`setup.py`的配置，又是另外一个话题了。更多选项参考文档。

* 项目中的实例

    [传送门](https://github.com/YabZhang/tornado_base)

```
git clone https://github.com/YabZhang/tornado_base
cd torando_base

# install && buildout
python bootstrap.py

# start server
bin/server
```

然后，就可以滚去写代码了...

### the end

`buildout`适用于自动化构建，能够有效提高工作效率。推荐与`virtualenv`一起使用。
功能上与`docker`和`make`有所重叠，好在简单快捷，也能很好地满足中小项目的需要，没做过大项目-.-;

