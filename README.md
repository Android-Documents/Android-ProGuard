# Android分享：代码混淆那些事

## 前言
[ProGuard](http://proguard.sourceforge.net/)是一个开源的Java代码混淆器。它可以混淆Android项目里面的java代码，对的，你没看错，仅仅是java代码。它是无法混淆Native代码，资源文件drawable、xml等。

## ProGuard作用
* 压缩: 移除无效的类、属性、方法等
* 优化: 优化字节码，并删除未使用的结构
* 混淆: 将类名、属性名、方法名混淆为难以读懂的字母，比如a,b,c

## 混淆注意事项
### 不能混淆
	* 在AndroidManifest中配置的类，比如四大组件
	* JNI调用的方法
	* 反射用到的类
	* WebView中JavaScript调用的方法
	* Layout文件引用到的自定义View
	* 一些引入的第三方库（一般都会有混淆说明的）。这里推荐两个开源项目，里面收集了一些第三方库的混淆规则：
		- android-proguard-snippets
		- android-proguard-cn
		
> 不难理解，混淆之后，类名会变成a,b,c这种，通过包名+类名自然就会找不到该类了，自然就会出现ClassNotFoundException异常。这里推荐一篇文章：
[http://www.itnose.net/detail/6043297.html](http://www.itnose.net/detail/6043297.html)

### Log处理

我们都知道，使用Log的时候，需要用到TAG，然而TAG我们一般都会写成： 

```
private static final String TAG = MainActivity.class.getSimpleName()
```

这时候MainActivity如何被混淆的话，log输出信息就会变成V/a:xxxxxxx，所以为了让log输出信息维持原状，可以将TAG处理成固定的字符串： 

```
private static final String TAG = "MainActivity"
```

正好Android Studio里面的Live Templates
![image](https://github.com/Android-Documents/Android-ProGuard/blob/master/imgs/android-studio-live-templete.jpg)

能让你轻轻松松的声明TAG
![image](https://github.com/Android-Documents/Android-ProGuard/blob/master/imgs/android-studio-logt.gif)

关于Log处理，推荐一篇文章：[https://www.zybuluo.com/shark0017/note/163330](https://www.zybuluo.com/shark0017/note/163330)

### Crash信息处理
代码混淆的时候记得加上在混淆文件里面记得加上这句： 

```
# keep住源文件以及行号 
-keepattributes SourceFile,LineNumberTable
```

否则你看到的崩溃信息就会变成这样子（图片来自bugly）
![image](https://github.com/Android-Documents/Android-ProGuard/blob/master/imgs/cash-by-bugly.png)
这里推荐bugly的一篇文章：[http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=26&extra=page%3D1](http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=26&extra=page%3D1)

### ProGuard使用
### 常用语法
#### 保留

* -keep {Modifier} {class_specification} 保护指定的类文件和类的成员
* -keepclassmembers {modifier} {class_specification} 保护指定类的成员，如果此类受到保护他们会保护的更好
* -keepclasseswithmembers {class_specification} 保护指定的类和类的成员，但条件是所有指定的类和类成员是要存在。
* -keepnames {class_specification} 保护指定的类和类的成员的名称（如果他们不会压缩步骤中删除）
* -keepclassmembernames {class_specification} 保护指定的类的成员的名称（如果他们不会压缩步骤中删除）
* -keepclasseswithmembernames {class_specification} 保护指定的类和类的成员的名称，如果所有指定的类成员出席（在压缩步骤之后）
* -printseeds {filename} 列出类和类的成员-keep选项的清单，标准输出到给定的文件

#### 压缩
* -dontshrink 不压缩输入的类文件
* -printusage {filename}
* -whyareyoukeeping {class_specification}

#### 优化
* -dontoptimize 不优化输入的类文件
* -assumenosideeffects {class_specification} 优化时假设指定的方法，没有任何副作用
* -allowaccessmodification 优化时允许访问并修改有修饰符的类和类的成员

#### 混淆

* -dontobfuscate 不混淆输入的类文件
* -obfuscationdictionary {filename} 使用给定文件中的关键字作为要混淆方法的名称
* -overloadaggressively 混淆时应用侵入式重载
* -useuniqueclassmembernames 确定统一的混淆类的成员名称来增加混淆
* -flattenpackagehierarchy {package_name} 重新包装所有重命名的包并放在给定的单一包中
* -repackageclass {package_name} 重新包装所有重命名的类文件中放在给定的单一包中
* -dontusemixedcaseclassnames 混淆时不会产生形形色色的类名
* -keepattributes {attribute_name,…} 保护给定的可选属性，例如LineNumberTable, LocalVariableTable, SourceFile, Deprecated, Synthetic, Signature, and InnerClasses.
* -renamesourcefileattribute {string} 设置源文件中给定的字符串常量

#### 通配符匹配规则
通配符               | 规则
------------------- | ------------------------------------
？                  | 匹配单个字符
*                   | 匹配类名中的任何部分，但不包含额外的包名
\*\*                | 匹配类名中的任何部分，并且可以包含额外的包名
%                   | 匹配任何基础类型的类型名
*                   | 匹配任意类型名 ,包含基础类型/非基础类型
...                 | 匹配任意数量、任意类型的参数
<init>              | 匹配任何构造器
<ifield>            | 匹配任何字段名
<imethod>           | 	匹配任何方法
*(当用在类内部时)     | 匹配任何字段和方法

更详细的语法请戳:[http://proguard.sourceforge.net/manual/usage.html#classspecification](http://proguard.sourceforge.net/manual/usage.html#classspecification)

#### Android Studio中使用方法

按照上面的语法规则编写<strong>proguard-rules.pro</strong>后，需要在<strong>build.gradle</strong>中配置，需要混淆的时候，设置<strong>minifyEnabled=true</strong>即可

全选复制放进笔记

```
buildTypes {
    debug {
        minifyEnabled false
    }
    release {
        signingConfig signingConfigs.release
        minifyEnabled true
        proguardFiles 'proguard-rules.pro'
    }
}
```

#### ProGuard的输出文件说明

混淆后，会在/build/proguard/目录下输出下面的文件

- dump.txt 描述apk文件中所有类文件间的内部结构。
- mapping.txt 列出了原始的类，方法，和字段名与混淆后代码之间的映射。
- seeds.txt 列出了未被混淆的类和成员
- usage.txt 列出了从apk中删除的代码 
当我们需要处理crash log的时候，就可以通过mapping.txt的映射关系找到对应的类，方法，字段等。方法如下：

```
sdk\tools\proguard\bin 目录下有个retrace工具可以将混淆后的报错堆栈解码成正常的类名
window下为retrace.bat，linux和mac为retrace.sh. 使用方法如下：
	1. 将crash log保存为yourfilename.txt
	2. 拿到版本发布时生成的mapping.txt
	3. 执行命令retrace.bat -verbose mapping.txt yourfilename.txt
```

所以我们每次打包版本都需要保存最新的mapping.txt文件。如果要使用到第三方的crash统计平台，比如bugly，还需要我们上传APP版本对应的mapping.txt.每次都要保存最新的mapping文件，那不就很麻烦？放心，gradle会帮到你，只需要在bulid.gradle加入下面的一句。每次我们编译的时候，都会自动帮你保存mapping文件到本地的。

```
android {
applicationVariants.all { variant ->
        variant.outputs.each { output ->
            if (variant.getBuildType().isMinifyEnabled()) {
                variant.assemble.doLast{
                        copy {
                            from variant.mappingFile
                            into "${projectDir}/mappings"
                            rename { String fileName ->
                                "mapping-${variant.name}.txt"
                            }
                        }
                }
            }
        }
        ......
    }
}

```

### 参考
- [https://blog.gmem.cc/proguard-study-note](https://blog.gmem.cc/proguard-study-note)
- [http://developer.android.com/intl/zh-cn/tools/help/proguard.html](https://blog.gmem.cc/proguard-study-note)


** <b>本文转载自[https://segmentfault.com/a/1190000004461614](https://segmentfault.com/a/1190000004461614)</b>
