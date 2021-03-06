# APP开发框架

本文主要介绍一种APP开发框架，包含从UI到底层的整个框架。如果你准备做一个新APP，此框架可以让你快速入手，如果你是准备重构APP代码，此框架可以带给你一些重构的方法。

代码地址：
[iOS框架代码](https://github.com/hamilyjing/JJAppFramework/tree/master/JJiOSFramework)
[Android框架代码](https://github.com/hamilyjing/JJAppFramework/tree/master/JJAndroidFramework)

## 框架简介

![image](https://mmbiz.qlogo.cn/mmbiz/YTAjOycganMibYRRc0zeYQSywpN6kTvHYeLH01iahUPyxQxf1xTy5mu0V4ibJatzQphSSvM97rBr2k5IgicQgpayAA/0?wx_fmt=png)

框架分三层，由下至上：通用层，服务层，视图层。

1. 通用层

	通用层包含第三方库和通用工具，这些工具和业务无任何关系，并且可以单独拿出来放到任何工程中。
	
2. 服务层
	
	服务层由服务工厂管理所有服务组件，对外提供装载、卸载和获取服务接口。服务组件采用插件模式，满足特定协议（代码中新增组件需要继承JJService，并且实现serviceName方法）的组件就可以放到服务层，由服务工厂进行管理。
	
	服务组件处理业务逻辑，组件之间不要有任何依赖关系（除少数组件被依赖，如登录服务，其他组件需要登录信息），也不要有任何UI代码。服务组件由不同的功能集合（FeatureSet）组成，具体的业务逻辑写在每个功能集合中，数据模型也是保存在功能集合中。
	
	例如，我们新写一个信用卡服务，分了账单、还款等功能集合，账单功能集合中处理未出账单和已出账单业务逻辑。
	
3. 视图层

	视图层使用MVP模式开发每个功能模块，MVP模式达到视图和业务逻辑分离。
	
	对于相同的模块，在不同设备上，View可以不同，Presenter是可以复用。
	
	Presenter提供业务接口给View使用，Presenter可以调用不同的服务组件来满足View的需求，并将服务组件返回的结果回调给View。除非View的需求需要在UI层创建新的模型，其他情况可以直接使用服务组件中的数据模型。

## 框架详解

#### 数据回调方式

目前有三种方式可供选择，代理、回调函数和通知。代理是一对一（也可以做到一对多，只需要做一个代理容器即可），回调函数是一对一，通知是一对多，因此好的框架需要提供一对一和一对多两种回调方式，并提供统一的回调接口（详见JJService中callback函数）。

iOS框架提供了3种回调方式，Android框架提供回调函数和通知。

#### 服务组件的装载和卸载

使用者使用某一项服务组件时，服务工厂查找是否以装载过此组件，如果是，则将此组件返回给使用者，不是，则通过服务组件名称动态装载此组件。

不再使用的服务组件就需要卸载，卸载的方式有两种，一种是主动卸载（使用者调用服务工厂卸载API），另一种是自动卸载，当服务组件的代理个数为0且请求完成个数为0时才会被卸载。

#### 网路

框架实现了HTTP请求，iOS使用第三方库YTKNetwork，并对其进行了扩展（详见JJRequest），其中加入了网络应答数据自动转化为模型，模型的存储，以及对模型的操作。Android使用第三方库android-async-http，并参考YTKNetwork设计理念开发网络基础类（详见JJRequest）。

#### 数据存储

框架将HTTP应答数据自动映射为数据模型，iOS使用第三方库YYModel，Android使用第三方库fastJSON。

框架采用对象序列化后存储在文件中，没有采用数据库的方式。一是模型的修改需要需要修改数据库表，二是使用数据库达到数据流到模型映射的自动化难度较大，文件存储就不存在这些问题，而且对象的增删查效率可以满足大部分APP。

框架对存储的数据进行了加密。

## 其他

#### 代码规范

一个好的开发框架必然有一套代码规范准则，开发者必须严格遵守。好的代码规范可以提高review效率，降低维护和学习成本。

有了规范基础，我们可以写一些代码自动生成插件，提高开发效率。

#### 发布服务

由于服务层严格按照高内聚低耦合的规则，很容易发布一个或几个组件，其中发布方式有以下几种：

1. 组件单独发布
2. 组件+Presenter一起发布，使用者需要写视图
3. 组件+UI一起发布，使用者只需要调用我们的视图即可
