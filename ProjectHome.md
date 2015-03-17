# 介绍 #

> 这个框架的初衷，是为了方便让程序模块化、插件化，将一个apk应用拆分为多个apk。

> 不明白这个插件化、模块化是怎么回事的话，可以看看腾讯微信的安卓客户端中的插件配置。

> 在这里我会以腾讯微信为例，如何使用这个框架。
> (**腾讯微信并不是真正的插件化，它是伪的，插件并非与它的主程序分离开，结果就是每次插件的更新，都必须以整个程序的更新为代价**)

# 都能干啥 #

> 框架的思想，主要是通过apk清单文件中的sharedUserId属性来吧多个apk融合为单一的dalvik虚拟机，也就是融合为一个进程，这样就变相逾越了android框架中不同apk权限不同无法互通的鸿沟(rpc啦什么的其它的毕竟不如这个来的实在)。

> 从最简单的皮肤插件到复杂的涉及数据库的拓展功能，从普通activity跳转到把插件的activity转变为view并附加到主程序中的拓展功能，都是可以通过本框架来实现的，当然，本框架只是提供了一种途径，如何编码还得靠自己。

> 本人能力有限，可能有一些错误、疏漏或者不足之处，请不吝指教，我的博客地址在左下角，或者我的邮箱，都可以联系到我。

> ps:下面会以腾讯微信为例，这里声明一下，本人没有对微信安卓客户端进行任意形式的反编译与破解工作，只是看球半天进行简单的猜测而已。

# 框架结构 #

> 本插件框架由三个包组成：

  * org.igeek.android.pluginframework  **_这个包种类是框架的主要操作类_**
    1. PluginBuilder  **_组装插件_**
    1. PluginDescription  **_组装用户自定义的插件描述_**
    1. PluginInvoke  **_插件功能调用_**
    1. PluginSearch  **_查找插件_**
  * org.igeek.android.pluginframework.annotation  **_未来可能会考虑使用注解_**
    1. PluginDescription  **_用于用户自定义描述类的注解_**
    1. PluginFeature  **_用于插件类的注解_**
    1. PluginMethod  **_用于插件类方法的注解_**
  * org.igeek.android.pluginframework.beans  **_框架内部使用的一些beans_**
    1. Plugin  **_插件包_**
    1. PluginFeature  **_插件类_**
    1. PluginFeatureMethod  **_插件方法_**
  * org.igeek.android.pluginframework.util  **_一些工具类_**
    1. XMLParse  **_插件包_**


# 如何使用 #

> 以腾讯微信为例，虽然从表面来看，它是插件化的多种功能的一种集成。_但是，这都是虚的，每次更新时你会发现，哪怕一个小小的功能添加，都会让用户更新整个程序，截至目前(2011-12-17)，微信让我更新到3.5，修复了一些小bug，增加了1个插件功能，以及其它的小改进，就得让我下载并升级整个程序，6.8m啊。。_

> 在工程开发时，预留插件的使用、管理页面，如：

> ![http://android-application-plug-ins-frame-work.googlecode.com/svn/trunk/%20android-application-plug-ins-frame-work/trunk/AndroidPluginFramework/srcshoot/weixin1.png](http://android-application-plug-ins-frame-work.googlecode.com/svn/trunk/%20android-application-plug-ins-frame-work/trunk/AndroidPluginFramework/srcshoot/weixin1.png)
这是微信的插件浏览页面，所有可暴露在这里的插件都会被列出来

> ![http://android-application-plug-ins-frame-work.googlecode.com/svn/trunk/%20android-application-plug-ins-frame-work/trunk/AndroidPluginFramework/srcshoot/weixin2.png](http://android-application-plug-ins-frame-work.googlecode.com/svn/trunk/%20android-application-plug-ins-frame-work/trunk/AndroidPluginFramework/srcshoot/weixin2.png)
> 这是微信的插件管理界面，可以卸载安卓，但这些都是虚的，可能只是把服务关了，并非真正的卸载


> ## 主程序(APK) ##

  * 、主程序清单

> 首先为工程想好一个名字，并在清单中加入sharedUserId属性，属性值就是想好的那个名字，例如微信：com.qq.weixin
```
   android:sharedUserId="com.qq.weixin"
```


  * 、为插件描述写一个类

> ![http://android-application-plug-ins-frame-work.googlecode.com/svn/trunk/%20android-application-plug-ins-frame-work/trunk/AndroidPluginFramework/srcshoot/weixin2.png](http://android-application-plug-ins-frame-work.googlecode.com/svn/trunk/%20android-application-plug-ins-frame-work/trunk/AndroidPluginFramework/srcshoot/weixin2.png)
> 这就是插件描述

> 首先在主程序中单独建立一个包，并写一个类，这个类相当于一个bean，具有相应的插件描述信息的字段，如微信，那么这个bean的字段就有 **插件名**:_漂流瓶_   **插件logo**:_那个人头_   **插件描述**:_描述信息..._

> 这个类在主程序中不会很多的使用，只会读取框架返回的插件的bean。


  * 、引入本框架的jar包或导入源码

> 正式编码时，可在任何地方调用框架。具体步骤是这样的

> _1、查找插件_
> 本框架是根据sharedUserId来查找插件。它首先会根据自身apk的sharedUserId值来查找系统中具有相同属性的包并加载。
  * ��以说，插件apk的sharedUserId值也应与主程序相同

```
    	//首先，就是查找插件
    	PluginSearch psearch=new PluginSearch();
    	//第一次获得的是简要的插件描述
    	List<Plugin>  plugins=psearch.getPlugins(this);
    	
    	//然后将插件再组装一下
    	PluginBuilder pbuilder=new PluginBuilder(this);
    	//将用户所定义插件描述融合进去
    	plugins=pbuilder.buildPluginsDescrition(plugins);
```

> 这样就把所有插件找到手了，很简单吧？

> _1、调用插件_
> 找到插件后肯定是调用，最简单应用，就是靠Button按钮来调用

> 比如微信的摇一摇，点了它就触发框架的插件调用
```
	@Override
	public void onClick(View v) {
		//新建一个插件调用类
		PluginInvoke pi=new PluginInvoke(AndroidPluginFrameworkActivity.this);
	
		
		//这是真正的调用
		//三个参数，第一个是Plugin类型，第二个是PluginFeature，第三个PluginFeatureMethod
		pi.invoke(plug, pf, fm);
		
	}
	});
```


> ## 插件程序(APK) ##

  * 、插件程序清单

> 同主程序的清单文件一样，sharedUserId必须有且和主程序的一样
> 插件的activity的意图过滤描述必须是这样
> 示例：
```
  <intent-filter>
                <category android:name="android.intent.category.DEFAULT" />
  </intent-filter>
```

  * 、插件程序的功能**> 插件可以包含任意类，如同一个普通的apk工程一样。**

> 但是，向主程序提供调用的类，必须是个activity，方法必须是无返回值的，有且只有一个Context参数，框架会反射这个方法，并将主程序的context句柄传入，也就是，不管你用不用这个context，他都是被框架传入的。

> 如果主程序规定了描述类，也就是框架中需要PluginDescription来调用的bean，在插件工程中，也必须有主程序这个bean，包名也必须一样。插件程序可以直接继承或者更改这个bean的字段，方便主程序去获取这个插件的描述信息。这种方式，是避过框架，给主程序一种方便。

  * 、插件程序的plugin.xml配置

> http://android-application-plug-ins-frame-work.googlecode.com/files/plugin.xml
> plugin.xml不能改名，必须放在工程的 assets 目录下。

> 内容大致是这样：
```
<?xml version="1.0" encoding="UTF-8"?>
<!-- 这个xml配置文件放在插件工程的 assets 目录下 -->


<!-- 插件提供的功能(类) -->
<plugin-features>

	<!-- 描述类，这个是自定义的 -->
	<!-- 这个描述类使用户定义的，在主程序中必须有，插件工程中也必须有，并被继承 -->
	<description name="org.pakage.name.and.description.name"/>
	
	<!-- 这是一个功能(类)，必须是activity的子类 -->
	<feature name="org.pakage.name.and.activity.name1">
	
		<!-- 这是一个方法 -->
		<!-- name 方法名 -->
		<method need-context="true" name="methodName1" >描述信息</method>
		<method need-context="true" name="methodName2" >描述信息</method>
		
	</feature>
	
	<feature name="org.pakage.name.and.activity.name2">
		<method need-context="true" name="methodName1" >描述信息</method>
	</feature>
</plugin-features>
```

> 按着上面的套路来配置，就很容易了。


# 最后 #

> 上面说的比较泛，最好还是看看示例工程，里面包含了一个主程序和两个插件扩展，总共是4个工程，也就是说，在用户端是4个apk
> http://android-application-plug-ins-frame-work.googlecode.com/files/demo.zip