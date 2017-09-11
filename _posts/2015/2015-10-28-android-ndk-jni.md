---
layout: post
comments: true
title: "关于android ndk的jni总结"
description: "关于android ndk的jni总结"
categories: ["android", "ndk", "jni"]
---

## 开发工具支持

主要要点如下，更详细的应该参考官方文档:

1. 需要下载android ndk，并设置ANDROID_NDK_HOME并设置PATH, 在eclipse中顺便也设置一下。
2. eclipse支持android ndk开发，只需要在项目中右键添加Android Tools > Add Native Support即可。

## 配置文件

主要配置文件有2个: Android.mk,Application.mk,详细配置还是应该阅读官方文档。下面说一下常用配置。

#### Application.mk

详细配置参考https://developer.android.com/intl/zh-cn/ndk/guides/application_mk.html

* APP_STL := stlport_static 设置是否依赖的C++标准库特性，非常重要，详细参数参考https://developer.android.com/intl/zh-cn/ndk/guides/cpp-support.html#runtimes
* APP_ABI := armeabi armeabi-v7a 设置需要生成so的平台，可以指定或者用all
* APP_OPTIM := release 生成debug还是relase版本，默认就是release

#### Android.mk

详细配置参考https://developer.android.com/intl/zh-cn/ndk/guides/android_mk.html  
这个配置是可以一次性生成多个so文档的，只需要区分不同的LOCAL_MODULE、LOCAL_SRC_FILES即可。

**发现ndk好像默认不支持c/c++混编，所以最好统一成cpp后缀。又或者是我不清楚实际是可以的**

* LOCAL_LDLIBS += -L$(SYSROOT)/usr/lib -lz -llog -landroid 依赖的库，这个例子表示依赖zlib、android log模块及android运行库
* LOCAL_MODULE    := protect 生成的模块名
* LOCAL_SRC_FILES := NativeApplication.cpp NativeHelper.cpp arcfour.cpp MultiDex.cpp 就是把so需要的相关源文件列出来

## jni编程

剩下的内容和Android都没特别关系了，都是java jni的知识。
对于android中jni的各种限制，可以参考官方文档： http://developer.android.com/intl/zh-cn/training/articles/perf-jni.html

#### 生成native方法的头文件

和普通java的没区别，用javah就可以了，就是需要在classpath中添加android的jar即可。举例:

```bash
cd native
javah -cp ./bin/classes;D:\05programs\Android\android-windows\platforms\android-19\android.jar -d ./jni com.huawei.g3.proxy.NativeApplication
```

默认生成的方法名是有特殊命名规则的(具体规则请自行查阅资料)，如果需要不同名字，可以在JNI_OnLoad中进行动态注册，参考如下：

```c++
void load(JNIEnv * env, jclass clz, jobject obj);
void run(JNIEnv * env, jclass clz, jobject obj);
static JNINativeMethod methods[] = { { "load", "(Landroid/app/Application;)V", (void*) load }, { "run", "(Landroid/app/Application;)V",
        (void*) run } };

jint JNI_OnLoad(JavaVM* vm, void* reserved) {
    JNIEnv* env;
    if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) {
        return JNI_ERR;
    }

    // Register methods with env->RegisterNatives.
    int len = sizeof(methods) / sizeof(methods[0]);
    jclass native = env->FindClass("com/huawei/g3/proxy/NativeApplication");
    if (env->RegisterNatives(native, methods, len) < 0) {
        return JNI_ERR;
    }
    return JNI_VERSION_1_6;
}
```

#### 调用java对象、方法、属性

当需要和java进行交互的时候，需要通过特定的api进行调用(类似java的反射，比较麻烦)，这种方式是绕过java安全检查机制的。

首先，需要了解jni中的类型表示法(基本就是java字节码那套表示法)，**特别注意的是内部类的写法**

| Type Signature            | Java Type             | 备注                              |
|---------------------------|-----------------------|-----------------------------------|
| Z                         | boolean               |                                   |
| B                         | byte                  |                                   |
| C                         | char                  |                                   |
| S                         | short                 |                                   |
| I                         | int                   |                                   |
| J                         | long                  |                                   |
| F                         | float                 |                                   |
| D                         | double                |                                   |
| L fully-qualified-class ; | fully-qualified-class | Ljava/lang/String; 或内部类Lcom/test/A$B; |
| [ type                    | type[]                | [I 或 [Ljava/lang/String;                                  |
| ( arg-types ) ret-type    | method type           | ()V 或 (Ljava/lang/String;I)Z                                  |
| V                         | void                  |                                   |

看懂上面一套表示法，下面的代码也比较容易理解了:

```c++
jclass contextClass = env->FindClass("android/content/Context");
jfieldID fieldID = env->GetStaticFieldID(contextClass, "MODE_PRIVATE", "I");
jint mpFv = (jint) env->GetStaticIntField(contextClass, fieldID);

jstring _payload_dex = env->NewStringUTF("payload_dex");

jclass appClass = env->FindClass("android/app/Application");
jmethodID methodID = env->GetMethodID(appClass, "getDir", "(Ljava/lang/String;I)Ljava/io/File;");
jobject dex = env->CallObjectMethod(obj, dirMd, _payload_dex, mpFv); //obj是Application对象，传进来的
```

**需要注意的时候，FindClass参数中类名是不以L开头，不以;结束的**

上面的代码，其实就完成了下面一句java代码:

```java
File dex = obj.getDir("payload_dex", Context.MODE_PRIVATE);
```

基本套路都是一样的： 找到类、找到方法或属性、调用方法或调用属性，对应的是jclass、jmethodID、jfieldID几种类型。  
详细的方法应该参考jni的官方文档，也很好理解。

```c++
* CallStatic<type>Method
* Call<type>Method
* SetStatic<type>Field
* Set<type>Field
* GetStatic<type>Field
* Get<type>Field
```

这里主要讲一下注意点:

* 不带后缀、带V、带A的方法名有什么区别

以CallObjectMethod为例，会存在三个方法:  CallObjectMethod, CallObjectMethodV, CallObjectMethodA
这个方法都是返回Object对象(jobject)的，效果是没什么区别的，只在于参数传递机制上存在区别。

* 类型能不完全匹配么?

像java反射那样，获取方法是可以不指定参数类型的。但是jni的类型是必须完全匹配的，
例如找方法void get(HashMap map)的时候，需要使用"(Ljava/util/HashMap;)V", 而不能使用"(Ljava/util/Map;)V"。

这种问题在处理api兼容性的时候就特别突出。例如下面的代码:

```c++
    if (version < 19) {
        //using hashmap
        ft1 = "Ljava/util/HashMap;";
        ft2 = "java/util/HashMap";
    } else {
        //using arraymap
        ft1 = "Landroid/util/ArrayMap;";
        ft2 = "android/util/ArrayMap";
    }

    jobject mPackages = _getField(env, currentActivityThread, "mPackages", ft1);
    ...    
    jmethodID methodID = _getMethod(env, ft2, "get", "(Ljava/lang/Object;)Ljava/lang/Object;");
    jobject wr = env->CallObjectMethod(mPackages, methodID, (jobject) pk);
```

在java中就可以不用那么烦躁，获取mPackages对象，强制转换成两个类的共同接口Map即可，省事很多。

* 关于引用、字符串、异常处理

在jni中主要有LocalRef、GlobalRef两种。正常产生的jobject对象，是属于LocalRef的，它的生命周期在当前线程的当前方法有效，类似于c/c++在栈分配的对象。   
**官方tips也提到了，即使这个对象本身在本地方法返回之后仍然存在，这个引用也是无效的。而实际上只预留了16个LocalRef空间**  

所以在使用上需要特别注意:

1. 不要过度分配LocalRef，及时通过DeleteLocalRef方法进行删除。或者通过EnsureLocalCapacity/PushLocalFrame预留更多，不过貌似很少需要。
2. 如果需要在多次调用中保留，应该采用GlobalRef。通过NewGlobalRef/DeleteGlobalRef手动维护引用。
3. 和反射一样，查找类、获取方法、获取属性都是有消耗的，在频繁调用的jni方法中，应该通过GlobalRef预先保留相关对象。
4. 对于stirng类，如果和原生c字符串进行转换操作的时候，需要注意释放内存。
5. 虽然C++本身也有异常处理，但是切记空指针异常不同于java，需要注意可能为NULL的代码。
6. 不像java传参是传值(对象是隐含指针传递)，在c++中要注意区分传值、传指针、传引用。

#### 通过GlobalRef优化jni的例子

注意: jint等基本类型、jmethodID、jfieldID都不是jobject，不需要管理引用。
```c++
static jobject decryptCipher;

jint JNI_OnLoad(JavaVM* vm, void* reserved) {
        ...
	jobject localDecryptCipher = env->CallStaticObjectMethod(cipher, mid,
			desbuf);
	decryptCipher = (jobject) env->NewGlobalRef(localDecryptCipher);
	env->DeleteLocalRef(localDecryptCipher);
}

void JNI_OnUnload(JavaVM *vm, void *reserved) {
        ...
	if (decryptCipher != NULL) {
		env->DeleteGlobalRef(decryptCipher);
	}
	if (encryptCipher != NULL) {
		env->DeleteGlobalRef(encryptCipher);
	}
}
```

#### 操作java字符串、c字符串的例子

```c++
    const char *c_msg2 = env->GetStringUTFChars(dataDir, NULL);  // dateDir是jstring对象
    string libPath(c_msg2);
    env->ReleaseStringUTFChars(dataDir, c_msg2); // 和GetStringChars不同，GetStringUTFChars方法会分配内存并进行拷贝到c字符串，所以需要手动释放
    libPath.append("/lib");
    LOGI("lib path is %s", libPath.c_str());
    jstring libDir = env->NewStringUTF(libPath.c_str()); // 重新转换成jstring
```