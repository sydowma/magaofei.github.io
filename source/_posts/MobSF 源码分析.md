---
title: MobSF源码分析
date: 2018年08月17日11:08:13

---

## 介绍

MobSF(Mobile-Security-Framework-MobSF)，是一个基于 Django 的移动端安全扫描应用。GitHub 地址：<https://github.com/MobSF/Mobile-Security-Framework-MobSF>

 

下边打算简单分析一下源码

## 分析

#### 目录

```shell
.
├── DynamicAnalyzer #动态扫描 app
│   ├── tools  # 运行平台对应的 adb 工具和工具包等
│   └── views  # 功能函数
├── LICENSES 
├── MalwareAnalyzer  # 恶意软件扫描
├── MobSF   # 项目项目，设置等文件
│   ├── forms.py      # 上传文件接口表单
│   ├── kali_fix.sh
│   ├── models.py
│   ├── rest_api.py   # 对外 RESTAPI
│   ├── settings.py   # 配置文件
│   ├── urls.py       # URL 路由函数
│   ├── utils.py      # 通用的一些功能，比如寻找 Java，检查更新，获取 apiKey 等
│   ├── views.py      # 视图函数
│   └── wsgi.py 
├── StaticAnalyzer
│   ├── models.py   # ORM
│   ├── tests.py    # 单元测试
│   ├── migrations  # SQL 脚本
│   ├── test_files  # 测试文件
│   ├── tools       # 用到的工具包
│   └── views       # 功能实现函数
├── downloads
│   ├── 3857ea931f051b319b558488518e7c16-screenshots-apk # 动态扫描产生的截图等
├── install  
│   └── windows  # 扫描 Windows appx 
├── logs         # 日志
│   └── certs
├── scripts      # shell 脚本
│   └── cloud
├── static       # CSS JS 文件
│   ├── bootstrap
│   ├── css
│   ├── dash
│   ├── fonts
│   ├── img
│   ├── js
│   └── plugins
├── templates  # HTML 文件
│   ├── dynamic_analysis
│   ├── general
│   ├── pdf
│   └── static_analysis
└── uploads
    ├── 2ff4f3be34ff3b8763671b77790da91e #上传的文件

```



由此可见，源代码当中，需要关注的是 `MobSF` 、`StaticAnalyzer` 、`DynamicAnalyzer` ，而里边 `MobSF`主要是 Django 的一些设置和 API ，动态分析用的不多，所以最重要的还是 `StaticAnalyzer` 这个 App 需要重点去关注。



### Django 部分

刚开始部署的时候我发现很奇怪的一个地方，这个项目居然在源码中保留了 static 文件夹，一般 Django 项目到发布的时候才会生成这个文件夹，如果放到源代码管理中，势必要增大项目的体积。

还有一点就是 Log 打印的并不友好，代码中出现了大量的 `print` 语句，而不是 log 模块。

还有在 `rest_api` 的处理也可以看出，写的非常随意，表单效验没有，通篇的 IF_ELSE ，让人看了难受。知道作者是印度人后，我就知道了什么，我打算提交一些 `Pull Request` ，如果允许的话，我是愿意帮忙改改的。

#### ORM

数据库部分只有`StaticAnalyzer`部分用到，同样，这一部分也是最常用的功能

### 扫描部分

#### 静态分析

在对样本进行静态分析时，MobSF主要使用了现有的dex2jar、dex2smali、jar2java、AXMLPrinter、CertPrint等工具。其主要完成了两项工作： 解析AndroidManifest.xml得到了应用程序的各类相关信息、 对apk进行反编译得到java代码，而后利用正则匹配找出该样本主要进行了哪些工作。    

#### 动态分析

仅限于 Android，动态分析在真机运行需要 WiFiADB，速度慢，效果不理想

##### 原理

而在对样本进行动态分析时，MobSF主要利用到了Xposed框架、Droidmon实现对应用程序调用API的情况进行监控，并且可灵活维护一份需要hook的API列表。同时，MobSF还使用了DataPusher来对样本数据进行打包、使用了ScreenCast结合adb shell input完成对手机的远程控制功能。当然，其中还使用隐藏root权限、伪造成正式机器等技术来应对一些反虚拟机的程序。其主要做了一下几件事： 

1、利用webproxy实现代理进而拦截样本流量。 

2、安装证书以便拦截https流量。 

3、遍历所有activity，尽量多的获取各activity运行得到的日志。

 4、利用正则匹配出API及参数和返回值。

5、实时更新恶意url库，以url信息特征进行查杀。 





#### URLS

四部分

- `MobSF.views` 里边主要是HTML页面
- `StaticAnalyzer.views` Android 静态分析
- `DynamicAnalyzer.views` Android 动态分析
- `MobSF.rest_api` 对外的REST API (其实根本不 REST)

#### Views

`MobSF.views`这里边主要是一些 HTML 页面和部分功能(upload/delete)

`StaticAnalyzer.views` 静态扫描的功能函数



#### StaticAnalyzer

##### ORM

数据库写的很简单

- RecentScansDB
- StaticAnalyzerAndroid
- StaticAnalyzerIPA
- StaticAnalyzerIOSZIP
- StaticAnalyzerWindows

可以看出来，首先扫描之后会往 `RecentScansDB`写入数据，然后根据不同的文件往不同的库里写详细数据，两者是用MD5值来判断指定文件。

#### 功能实现

##### 扫描 API 的实现

1. `MobSF.rest_api.api_scan()`
2. `StaticAnalyzer.views.android.static_analyzer()`

在`static_analyzer()` 函数里边可以看到一些数据的获取，基本上是用字典来存取各类信息，这个信息由一个响应的的功能函数得到，而这个功能函数，你点进去看，会发现是执行的 Shell，在 Shell 输出的时候，会有一些正则等匹配方式，去拿到想要的数值，然后返回。最后程序写入库中。



#### Android 静态扫描

未完待续

#### iOS 静态扫描

未完待续

##### 其他 API

在`StaticAnalyzer.views` 里需要先重点看一下 `shared_func.py` 文件





#### 参考

```
1. http://purpleroc.com/MD/2016-08-31@Android%20Malware%20Analysis%20Tool(1)--MobSF.html
```

