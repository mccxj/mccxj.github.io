---
layout: post
comments: true
title: "jdk7编译的bug记录"
description: "jdk7编译的bug记录"
categories: ["jdk7", "bug"]
---

## 过程

昨天在编译某个android项目的时候，发现dex打包出错。
后来检查发现编译生成的SplashScreenActivity$1.class格式出错。

后来经常测试，发现jdk6正常，jdk7不正常，包括最新的u51版本。

## bug分析

相关的代码如下:

```java
private void checkUpdate() {
    if (UpgradeInfo.isApkLocalExist(SplashScreenActivity.this, UpgradeInfo.getFilePath())) {
        updateInstall();
    } else if (AUTO_UPDATE) {
        initDialog();
        mUpgradeInfo = new UpgradeInfo();
        mGetVersionConfig = new GetVersionConfig(SplashScreenActivity.this,
                new CheckUpgradeHandler(), mUpgradeInfo, getUpgradeRequestParam());
        mGetVersionConfig.start();
    } else {
        // /没版本且不升级
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                Intent i = new Intent(SplashScreenActivity.this, LoginActivity.class);
                startActivity(i);
                finish();
            }
        }, LOADING_TIME);
    }
}
```

正常情况下，Runnable匿名类会生成一个class，但是这里的AUTO_UPDATE被定义为static final，并且设置为true。
在这种情况下，jdk会优化掉后面的分支，但是jdk6不会生成class，但是jdk7会生成一个格式有误的class，反编译后如下：

```java
class SplashScreenActivity$1
  implements Runnable
{
  public void run();
}
```

所以，在进行dex打包的时候，就会检测到不规范的class，进行报错。
我想，如果在web应用上的话，应该是不会有问题的，因为这个类没有机会被使用。

后来，我发现在其他項目中也有类似写法，但是编译不会有问题。如下所示：

```java
if (UpgradeInfo.isApkLocalExist(LogoActivity.this,UpgradeInfo.getFilePath())) {
    // /检测到有新版本可以安装
    String msg = getString(R.string.dialog_check_version);
    DialogUtil.dialogForTwoButton(... , new OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    //..
                }
            });
} else if (AUTO_UPDATE) {
    // ....
} else {
    ///没版本且不升级
    new Handler().postDelayed(new Runnable() {

        @Override
        public void run() {
            Intent i = new Intent(LogoActivity.this, OperatorLoginActivity.class);
            startActivity(i);
            finish();
        }
    }, LOADING_TIME);
}
```


故猜测，这个bug只出现在第一个不被使用匿名类身上(因为这个类前面还有个OnClickListener的匿名类)。
经过测试，发现也的确是存在这样的情况。

## 规避方式

乖乖使用jdk6，不要使用jdk7。