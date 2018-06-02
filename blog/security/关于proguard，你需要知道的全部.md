![proguard流程](http://upload-images.jianshu.io/upload_images/2438937-204ad4d3ecac7c95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# proguard分为4个步骤：
* 压缩（shrink）
  移除未使用的类、方法、字段等；
* 优化（optimize）
  优化字节码、简化代码等操作；
* 混淆（obfuscate）
  使用简短的、无意义的名称重全名类名、方法名、字段等；
* 预校验（preverify）
  为class添加预校验信息。

# 一、4个步骤中的常量配置

### 1. 压缩（shrink）

**-dontshrink**
声明不进行压缩操作，默认情况下，除了-keep配置（下详）的类及其直接或间接引用到的类，都会被移除。

### 2. 优化（optimize）

**-dontoptimize**
不对class进行优化，默认开启优化。
**注意：**由于优化会进行类合并、内联等多种优化，-applymapping可能无法完全应用，**需使用热修复的应用，建议使用此配置关闭优化。**

**-optimizationpasses** *n*
执行优化的次数，默认1次，多次能达到更好的优化效果。

**-optimizations** [*optimization_filter*](https://www.guardsquare.com/en/proguard/manual/optimizations)
优化配置，可进行字段优化、内联、类合并、代码简化、算法指令精简等操作。
~~~
#只进行 移除未使用的局部变量、算法指令精简
-optimizations code/removal/variable,code/simplification/arithmetic

#进行除 算法指令精简、字段、类合并外的所有优化
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*
~~~

### 3. 混淆（obfuscate）

**-dontobfuscate**
不进行混淆，默认开启混淆。除-keep指定的类及成员外，都会被替换成简短、随机的名称，以达到混淆的目的。

**-applymapping** [*filename*](https://www.guardsquare.com/proguard/manual/usage#filename)
根据指定的mapping映射文件进行混淆。

**-obfuscationdictionary** [*filename*](https://www.guardsquare.com/proguard/manual/usage#filename)
指定字段、方法名的混淆字典，默认情况下使用abc等字母组合，比如根据自己的喜好指定中文、特殊字符进行混淆命名。

**-classobfuscationdictionary** [*filename*](https://www.guardsquare.com/proguard/manual/usage#filename)
指定类名混淆字典。

**-packageobfuscationdictionary** [*filename*](https://www.guardsquare.com/proguard/manual/usage#filename)
指定包名混淆字典。

**-useuniqueclassmembernames**
指定相同的混淆名称对应不同类的相同成员，不同的混淆名称对应不同的类成员。在没有指定这个选项时，不同类的不同方法都可能映射到a,b,c。
有一种情况，比如两个不同的接口，拥有相同的方法签名，在没有指定这个选项时，这两个接口的方法可能混淆成不同的名称。但如果新增一个类同时实现这两个接口，并且利用-applymapping指定之前的mapping映射文件时，这两个接口的方法必须混淆成相同的名称，这时就和之前的mapping冲突了。
**在某此热修复场景下需要指定此选项。**

**-dontusemixedcaseclassnames**
指定不使用大小写混用的类名，默认情况下混淆后的类名可能同时包含大写小字母。这在某些对大小写不敏感的系统（如windowns）上解压时，可能存在文件被相互覆盖的情况。

**-keeppackagenames** [*[package_filter](https://www.guardsquare.com/proguard/manual/usage#filters)*]
指定不混淆指定的包名，多个包名可以用逗号分隔，可以使用? * \*\*通配符，并且可以使用否定符（!）。

**-keepattributes** [*[attribute_filter](https://www.guardsquare.com/en/proguard/manual/attributes)*]
指定保留属性，多个属性可以用多个-keepattributes配置，也可以用逗号分隔，可以使用? * \*\*通配符，并且可以使用否定符（!）。
比如，在混淆ibrary库时，应该至少keep Exceptions, InnerClasses, Signature；如果在追踪代码，还需要keep符号表；使用到注解时也需要keep。
~~~
-keepattributes Exceptions,InnerClasses,Signature
-keepattributes SourceFile,LineNumberTable
-keepattributes *Annotation*
~~~

**-keepparameternames**
指定keep已经被keep的方法的参数类型和参数名称，在混淆library库时非常有用，可供IDE帮助用户进行信息提示和代码自动填充。

### 4. 预校验（preverify）

 **-dontpreverify**
指定不对class进行预校验，默认情况下，在编译版本为micro或者1.6或更高版本时是开启的。但编译成Android版本时，预校验是不必须的，配置这个选项可以节省一点编译时间。
（Android会把class编译成dex，并对dex文件进行校验，对class进行预校验是多余的。）

# 二、keep配置
**-keep** [[,*modifier*](https://www.guardsquare.com/proguard/manual/usage#keepoptionmodifiers),...] [*class_specification*](https://www.guardsquare.com/proguard/manual/usage#classspecification)
指定类及类成员作为代码入口，保护其不被proguard，如：
~~~
-keep class com.rush.Test
-keep interface com.rush.InterfaceTest
-keep class com.rush.** {
    <init>;
    public <fields>;
    public <methods>;
    public *** get*();
    void set*(***);
}
~~~
* class表示keep类或接口
* interface仅表示keep接口

######类名 通配符如下：

| 通配符 | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| ?      | 匹配单个字符，包名分隔符（.）除外                            |
| *      | 匹配除（.）外的任意字符                                      |
| \*\*   | 匹配任意字符（包含.），如com.rush.**匹配com.rush包下的所有类及其所有子包的类。 |

######字段和方法 通配符如下：

| 通配符    | 含义                              |
| --------- | --------------------------------- |
| <init>    | 匹配所有构造方法                  |
| <fields>  | 匹配所有字段                      |
| <methods> | 匹配所有方法                      |
| ?         | 匹配单个字符，包名分隔符（.）除外 |
| *         | 匹配除（.）外的任意字符           |

######类型 通配符如下：

| 通配符 | 含义                                                   |
| ------ | ------------------------------------------------------ |
| %      | 匹配原始类型，如int, boolean等                         |
| ?      | 匹配任意单个字符                                       |
| *      | 匹配除包名分隔符（.）外的任意字符                      |
| \*\*   | 匹配任意字符，包括包名分隔符（.）                      |
| \*\*\* | 匹配任意类型（原始类型、非原始类型、数组或非数组类型） |
| ...    | 匹配任意参数个数，任意参数类型                         |

其中类配置完整定义如下，其中[]表示可选：
~~~
[@annotationtype] [[!]public|final|abstract|@ ...] [!]interface|class|enum classname
    [extends|implements [@annotationtype] classname]
[{
    [@annotationtype] [[!]public|private|protected|static|volatile|transient ...] <fields> |
                                                                      (fieldtype fieldname);
    [@annotationtype] [[!]public|private|protected|static|synchronized|native|abstract|strictfp ...] <methods> |
                                                                                           <init>(argumenttype,...) |
                                                                                           classname(argumenttype,...) |
                                                                                           (returntype methodname(argumenttype,...));
    [@annotationtype] [[!]public|private|protected|static ... ] *;
    ...
}]
~~~

keep过于简单粗暴，proguard提供了6种不同的配置：

| 保留                           | 防止被移除或重命名      | 防止被重命名（未使用的会被移除） |
| ------------------------------ | ----------------------- | -------------------------------- |
| 类和类成员                     | -keep                   | -keepnames                       |
| 仅类成员                       | -keepclassmembers       | -keepclassmembernames            |
| 如类含有某成员，保留类及其成员 | -keepclasseswithmembers | -keepclasseswithmembernames      |

# 三、其它常用配置

**-verbose**
指定在混淆过程中输出更多信息，配置这个选项后，在遇到异常时，将输出完整的堆栈，而不仅仅是异常消息。

**-dontnote** [*[class_filter](https://www.guardsquare.com/proguard/manual/usage#filters)*]
指定不输出潜在的错误或者遗漏，比如拼写错误或者缺失了有用的信息。class_filter是一个正则表达式，匹配到类将不输出这些信息。

**-dontwarn** [*[class_filter](https://www.guardsquare.com/proguard/manual/usage#filters)*]
指定一组类，不警告这些类中找不到引用或其它重要的问题。这个选项是很危险的，比如，找不到引用的错误可能导致代码不能正常work。
（在引用一些存在警告的jar包，这个选项比较有用。）

**-ignorewarnings**
指定输出所以警告信息，但继续进行混淆。同上一选项，慎用。

**-printconfiguration** [[*filename*](https://www.guardsquare.com/proguard/manual/usage#filename)]
指定输出整个过程中的所有配置，输出到标准输出流或者指定文件中。这有时候在调度配置时有用。

**-dump** [[*filename*](https://www.guardsquare.com/proguard/manual/usage#filename)]
指定在任一处理过程后，输出class文件的结构，可以输出到标准输出流或者指定文件中。

# 四、proguard配置示例

### 4.1 Android默认推荐配置
在IDE自动生成的project.properties文件中，有这样一行：
~~~
#proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt:proguard-project.txt
~~~
Android Studio默认生成的build.gradle文件有如下配置：
```
buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
}
```
其中getDefaultProguardFile('proguard-android.txt')获取的也是tools/proguard/proguard-android.txt。下面看一下这个文件的配置：
~~~
# 不使用大小写混合类名
-dontusemixedcaseclassnames
# 不路过引用库中的非public类
-dontskipnonpubliclibraryclasses
# 输出更多信息
-verbose

# 不进行优化
-dontoptimize
# 不进行预校验
-dontpreverify

# keep注解
-keepattributes *Annotation*
#keep google license服务接口
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# keep native方法及其所属类
-keepclasseswithmembernames class * {
    native <methods>;
}

# keep自定义view的get/set方法
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

# keep继续自Activity中所有包含public void *(android.view.View)签名的方法，如onClick
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

# keep枚举中的values和valueOf方法
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# keep Parcelable的CREATOR成员
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

# keep R文件的静态字段
-keepclassmembers class **.R$* {
    public static <fields>;
}

# 不输出support包中的警告
-dontwarn android.support.**
~~~

### 4.2 一个典型library库的配置
示例引用自[官方文档samples](https://www.guardsquare.com/en/proguard/manual/examples#library)
```
# 保存mapping映射文件到out.map
-printmapping out.map 

# keep已keep方法的参数类型及参数名称
-keepparameternames 
# 这个配置未弄清楚，待测试
-renamesourcefileattribute SourceFile 
-keepattributes Exceptions,InnerClasses,Signature,Deprecated,
                SourceFile,LineNumberTable,*Annotation*,EnclosingMethod 

# keep所有类的protected成员
-keep public class * { 
      public protected *; 
} 

# keep在jdk 1.2中编译器插入的代码
-keepclassmembernames class * { 
    java.lang.Class class$(java.lang.String); 
    java.lang.Class class$(java.lang.String, boolean); 
} 

# keep native方法
-keepclasseswithmembernames,includedescriptorclasses class * { 
    native <methods>; 
} 

# keep枚举中的values和valueOf方法
-keepclassmembers,allowoptimization enum * { 
    public static **[] values(); 
    public static ** valueOf(java.lang.String); 
} 

# keep系列化相关方法
-keepclassmembers class * implements java.io.Serializable { 
    static final long serialVersionUID; 
    private static final java.io.ObjectStreamField[] serialPersistentFields; 
    private void writeObject(java.io.ObjectOutputStream); 
    private void readObject(java.io.ObjectInputStream); 
    java.lang.Object writeReplace(); 
    java.lang.Object readResolve(); 
} 
```

### 4.3 一个典型Android App的配置
示例引用自[官方文档samples](https://www.guardsquare.com/en/proguard/manual/examples#androidapplication)
```
-dontpreverify 
-repackageclasses '' 
-allowaccessmodification 
# 不优化算法指令
-optimizations !code/simplification/arithmetic 
-keepattributes *Annotation* 

# keep继承自系统组件的类
-keep public class * extends android.app.Activity 
-keep public class * extends android.app.Application 
-keep public class * extends android.app.Service 
-keep public class * extends android.content.BroadcastReceiver 
-keep public class * extends android.content.ContentProvider
 
# keep自定义view及其构造方法、set方法
-keep public class * extends android.view.View { 
      public <init>(android.content.Context); 
      public <init>(android.content.Context, android.util.AttributeSet); 
      public <init>(android.content.Context, android.util.AttributeSet, int); 
      public void set*(...); 
} 

-keepclasseswithmembers class * { 
    public <init>(android.content.Context, android.util.AttributeSet); 
} 

-keepclasseswithmembers class * { 
    public <init>(android.content.Context, android.util.AttributeSet, int); 
} 

-keepclassmembers class * extends android.content.Context { 
    public void *(android.view.View); 
    public void *(android.view.MenuItem); 
} 

-keepclassmembers class * implements android.os.Parcelable { 
    static ** CREATOR; 
} 

-keepclassmembers class **.R$* { 
    public static <fields>; 
} 

# keep javascript注释的方法，使用到webview js回调方法的需要添加此配置
-keepclassmembers class * { 
    @android.webkit.JavascriptInterface <methods>; 
} 
```

# 五、关于反射

并不是所有会被反射引用的类都必须keep，在progurad过程中能直接分析到引用的类会被proguard做相应的处理：
```
# Class.forName的类名"SomeClass"被混淆后自动替换
Class.forName("SomeClass")
SomeClass.class
# 以下字段和方法名都会在被混淆后自动替换
SomeClass.class.getField("someField")
SomeClass.class.getDeclaredField("someField")
SomeClass.class.getMethod("someMethod", new Class[] {})
SomeClass.class.getMethod("someMethod", new Class[] { A.class })
SomeClass.class.getMethod("someMethod", new Class[] { A.class, B.class })
SomeClass.class.getDeclaredMethod("someMethod", new Class[] {})
SomeClass.class.getDeclaredMethod("someMethod", new Class[] { A.class })
SomeClass.class.getDeclaredMethod("someMethod", new Class[] { A.class, B.class })
AtomicIntegerFieldUpdater.newUpdater(SomeClass.class, "someField")
AtomicLongFieldUpdater.newUpdater(SomeClass.class, "someField")
AtomicReferenceFieldUpdater.newUpdater(SomeClass.class, SomeType.class, "someField")
```
写个demo验证下：
```
Class<?> clazz = Class.forName("com.rush.test.SimpleClass1");
clazz.getDeclaredMethod("Test1");
SimpleClass2.class.getDeclaredField("mTestField");
SimpleClass2.class.getDeclaredMethod("Test2");
```

对以上代码编译并proguard，结果如下：

```
Class.forName("com.rush.a.a").getDeclaredMethod("Test1", new Class[0]);
b.class.getDeclaredField("a");
b.class.getDeclaredMethod("a", new Class[0]);
```
* 通过Class.forName反射的class com.rush.test.SimpleClass1"被自动替换成了"com.rush.a.a"；
* 但通过Class.forName获取的class再去反射方法没有正确处理；
* 通过完整class.getDeclaredField或者getDeclaredMethod反射时能够把字段名和方法名自动替换掉。

从结果看，反射并不是大家想像的那样必须keep，proguard能自动分析到引用的情况都能正确处理。但有些类是在配置文件里配置，或者动态拼接类名反射的，这些情况需要做好keep。

**为了问题追踪的方便，建议所有会被反射引用的代码和library public接口都做好keep。**

# 六、关于proguard配置的一些建议

* 所有会被反射引用的类都做好keep（建议，虽然有些反射能被正确处理）。
  如native方法，四大组件，接口model，枚举，序列化类等。

* 只keep必须保留的内容，不要过度keep

* 使用热修复的App，添加-dontoptimize配置


# 七、参考文档：
* https://www.guardsquare.com/en/proguard/manual/introduction
* https://www.guardsquare.com/en/proguard/manual/usage
* https://www.guardsquare.com/en/proguard/manual/examples