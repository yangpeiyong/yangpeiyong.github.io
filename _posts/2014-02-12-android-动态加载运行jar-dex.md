---
layout:     post
title:      Android 动态加载运行jar/dex
date:       2014-02-12 11:21:29
summary:    DexClassLoader 可以从jar或者apk文件中加载类，执行相关代码。
categories: Android DexClassLoader
---

DexClassLoader 可以从jar或者apk文件中加载类，执行相关代码。

DexClassLoader 需要一个私有目录来缓存优化后的classes，注意不要将这个目录放在SD卡上。

 File optimizedDexOutputDir = context.getDir("dex", 0);
使用DexClassLoader来加载类：
{% highlight java linenos %}
DexClassLoader cl = new DexClassLoader(dexStoragePath.getAbsolutePath(),
                        optimizedDexOutputDir.getAbsolutePath(),
                        null,
                        getClassLoader());

Class libProviderClazz = null;

libProviderClazz = cl.loadClass( “com.example.dex.lib.LibraryProvider” );
LibraryInterface lib = (LibraryInterface) libProviderClazz.newInstance();
lib.showAwesomeToast(view.getContext(), “hello”);
{% endhighlight %}

这里使用了接口对loadClass进行了类型转化，当然也可以使用反射，不过反射是复杂和低效的。

总结一下，加载jar/apk中类的步骤如下：

1、获取jar/apk文件

2、通过DexClassLoader类加载器来解析优化jar/apk文件

3、通过loadClass函数来载入类

4、通过newInstance来生成类的对象，并使用

在下面的这两篇篇博客中详细介绍了DexClassLoader的使用。

http://yunfeng.sinaapp.com/?p=87

http://android-developers.blogspot.com/2011/07/custom-class-loading-in-dalvik.html

http://yunfeng.sinaapp.com/?p=87这篇文章中提到需要将jar/apk拷贝到私有目录再加载，经过测试，这是不需要的，jar/apk在SD卡中也可以。

Custom Class Loading in Dalvik | Android Developers Blog
