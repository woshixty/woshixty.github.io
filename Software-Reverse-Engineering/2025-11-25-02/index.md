# X Saver APK 拆包与合包实战：从 split APK 做回「胖 APK」

最近在分析一款叫 **X Saver** 的 Twitter 视频下载 App 时，我遇到的是 Play 商店常见的 **split APK** 形式，而不是传统的单一 APK。
本文记录一下完整的拆包、资源合并、重新打包的过程，以及中间踩过的坑，供之后遇到同类问题时参考。

---

## 一、背景：从 ADB 拿到的三个 APK

先通过 ADB 查到 X Saver 在设备上的安装路径：

```bash
adb shell pm path twittervideodownloader.twitter.videoindir.savegif.twdown
```

系统返回三条路径，大致长这样：

* `.../base.apk`
* `.../split_config.arm64_v8a.apk`
* `.../split_config.xxxhdpi.apk`

再用 `adb pull` 拉到本地即可。

这其实就是典型的 **Android App Bundle 拆分结果**：

* **base.apk**：应用主体（绝大部分逻辑代码 / 通用资源 / Manifest）
* **split_config.arm64_v8a.apk**：针对 `arm64-v8a` 的 so 库等 ABI 相关内容
* **split_config.xxxhdpi.apk**：针对某个屏幕密度（这里是 `xxxhdpi`）的图标、图片等资源

系统安装时会把它们视为一个 App 的不同 split 一起加载。

本文的目标是：**把这些内容合并成一个可单独安装的“胖 APK”来方便后续逆向分析和调试**。

---

## 二、工具与目录准备

使用的主要工具：

* **Apktool 2.12.1**：反编译、回编译资源和 Manifest
* 解压工具：7-Zip / unzip / Windows 资源管理器都可以
* JDK 自带的 `keytool`、`apksigner` 用于签名（或 Android SDK 中的 apksigner）

在工作目录下准备三个子目录（名字可按习惯）：

```text
XSaver/
  base.apk
  split_config.arm64_v8a.apk
  split_config.xxxhdpi.apk
```

先对 base 和 dpi split 做反编译：

```bash
apktool d base.apk -o base
apktool d split_config.xxxhdpi.apk -o split_dpi
```

ABI split 只包含 so 库，直接解压即可（也可以 apktool d，看个人习惯）：

```bash
mkdir split_abi
unzip split_config.arm64_v8a.apk -d split_abi
```

此时目录结构大致如下：

```text
XSaver/
  base/           # apktool 解包后的 base 工程
  split_dpi/      # apktool 解包后的 dpi split 工程
  split_abi/      # unzip 出来的 abi split 内容
```

后续所有修改都在 `base/` 目录上进行。

---

## 三、合并 ABI：拷贝 arm64-v8a 的 so 库

`split_config.arm64_v8a.apk` 的主要价值就是 `lib/arm64-v8a/*.so`。
直接把它们合并到 base 即可：

```bash
# 如果 base/lib/arm64-v8a 不存在先建目录
mkdir -p base/lib/arm64-v8a

# 复制 so
copy split_abi\lib\arm64-v8a\* base\lib\arm64-v8a\
```

（Windows 也可以用资源管理器直接拖；注意别把 `META-INF`、`AndroidManifest.xml` 之类的乱丢进 base）

---

## 四、合并 DPI：拷贝高分辨率资源

重点是 `split_dpi` 目录下的 `res` 内容。
我实际看到的结构类似：

```text
split_dpi/res/
  drawable-anydpi-v21/
  drawable-anydpi-v24/
  drawable-hdpi-v4/
  drawable-ldrtl-xxhdpi-v17/
  drawable-night-anydpi-v21/
  drawable-xhdpi-v4/
  drawable-xxhdpi-v4/
  values/
  values-hdpi/
  ...
```

> **关键点：这里是 apktool 解包后的 res，里面的 `.xml` 已经是「纯文本 XML」了，可以安全交给 aapt2 重新编译。**

如果你是直接 `unzip split_config.xxxhdpi.apk` 然后拷 `res/*.xml` 到 base，那里面的 XML 是 **二进制格式**，aapt2 会直接报错：

```text
error: not well-formed (invalid token).
```

这是我一开始踩的一个坑，所以正确做法一定是：**先 apktool d，再拷。**

拷贝命令示例：

```bash
# 各种 drawable 目录
copy /y /s split_dpi\res\drawable-anydpi-v21\*       base\res\drawable-anydpi-v21\
copy /y /s split_dpi\res\drawable-anydpi-v24\*       base\res\drawable-anydpi-v24\
copy /y /s split_dpi\res\drawable-hdpi-v4\*          base\res\drawable-hdpi-v4\
copy /y /s split_dpi\res\drawable-ldrtl-xxhdpi-v17\* base\res\drawable-ldrtl-xxhdpi-v17\
copy /y /s split_dpi\res\drawable-night-anydpi-v21\* base\res\drawable-night-anydpi-v21\
copy /y /s split_dpi\res\drawable-xhdpi-v4\*         base\res\drawable-xhdpi-v4\
copy /y /s split_dpi\res\drawable-xxhdpi-v4\*        base\res\drawable-xxhdpi-v4\

# values 相关
copy /y /s split_dpi\res\values\*       base\res\values\
copy /y /s split_dpi\res\values-hdpi\*  base\res\values-hdpi\
```

用资源管理器图形界面拷也完全可以，确认目标目录在 `base\res` 下即可。

---

## 五、Manifest 清理：去掉与 split 相关的声明

原始的 base Manifest 中会有一些声明，告诉 Play / 系统这是一个需要 split 的应用，例如：

```xml
<meta-data
    android:name="com.android.vending.splits.required"
    android:value="true"/>

<meta-data
    android:name="com.android.vending.splits"
    android:resource="@xml/splits0"/>
```

以及 `<manifest>` 标签上的 split 属性：

```xml
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:requiredSplitTypes="base__abi,base__density"
    android:splitTypes="">
```

在我们准备做「胖 APK」时，这些配置已经没意义了，反而有可能引发安装异常。
所以在 `base/AndroidManifest.xml` 中进行如下修改：

* 删除两条 `<meta-data>`：

  * `com.android.vending.splits.required`
  * `com.android.vending.splits`
* 从 `<manifest>` 标签上删除：

  * `android:requiredSplitTypes="base__abi,base__density"`
  * `android:splitTypes=""`

修改后 `<manifest>` 头部就恢复成比较正常的形式：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="twittervideodownloader.twitter.videoindir.savegif.twdown"
    android:versionCode="..."
    android:versionName="...">
```

---

## 六、public.xml 要不要动？

`public.xml` 位于 `res/values/public.xml`，主要记录「资源名 ↔ 资源 ID」。

这里的原则很简单：

> **只保留 base 自己的 `public.xml`，split 里的不管它，也不要拷。**

原因：

* 每个 APK（包括 split）本来都有自己的 `public.xml` 和 `resources.arsc`；
* 我们现在把 split 的资源文件（drawable / values）并进 base，让 aapt2 重新给它们分配 ID 即可；
* 如果把 split 的 `public.xml` 也合进去，很容易出现：

  * `resource xxx already defined`
  * ID 冲突等问题

因此在合并时：

* `base/res/values/public.xml` **保留原样**；
* 如果你从 `split_dpi` 的 `values` 里不小心把 `public.xml` 也拷过去，建议直接删掉它。

---

## 七、apktool.yml 中的 doNotCompress 调整

在 `base/apktool.yml` 中，可以看到类似配置：

```yaml
doNotCompress:
  - arsc
  - so
  - ...
```

有些版本的 apktool 可能会把 `dex` 加进 `doNotCompress`，这会导致回编译时产生奇怪问题。
在我的实践中，做了两点微调：

1. **删除 `- dex` 条目**（让 aapt2 正常处理压缩方式）
2. **添加 `- so`**（保持 native 库不被压缩）

最终 `doNotCompress` 里包含 `arsc`、`so` 等即可。
这一步不是绝对必须，但配合很多现有项目会更稳一点。

---

## 八、重新打包 & 常见错误排查

完成以上修改后，就可以尝试回编译：

```bash
apktool b base -o app-modified.apk
```

### 如果看到 `not well-formed (invalid token)`

这一般是：**你在 `base/res` 里面放进了“二进制 XML”**，而不是 apktool 解码过的文本 XML。

最常见的场景：

* 直接 `unzip split_config.xxxhdpi.apk`，从里面的 `res/drawable-anydpi-v21/*.xml` 拷到 base；
* 这些 `.xml` 是 aapt2 编译后的二进制格式，第一字节根本不是 `<`，aapt2 会直接报错。

解决方案就是前面提过的：

> **对 split APK 使用 `apktool d` 反编译，然后从反编译后的目录 `split_dpi/res` 拷资源。**

修正后重新 `apktool b`，就可以顺利生成 `app-modified.apk` 了。

---

## 九、重新签名并安装验证

`app-modified.apk` 是未签名的，需要用自己的证书重新签名：

1. 如果还没有 keystore，可先创建一个：

   ```bash
   keytool -genkeypair -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias x_saver_mod
   ```

2. 使用 apksigner 签名：

   ```bash
   apksigner sign \
       --ks my-release-key.jks \
       --ks-key-alias x_saver_mod \
       app-modified.apk
   ```

3. 安装到测试机：

   ```bash
   adb install -r app-modified.apk
   ```

如果前面合并没出错，一般可以正常安装并运行。
在我的实测中，合并后的 X Saver 功能正常，说明：

* so 库路径 / ABI 配置无问题
* 资源 ID 和 Manifest 也都被正确重编译

---

## 十、一些补充建议

1. **如果只是逆向分析，不一定非要合成胖 APK**

   * 完全可以：

     * 用 `jadx` 打开 `base.apk` 看 Java/Kotlin 逻辑；
     * 另开 IDA/Ghidra 看 `split_config.arm64_v8a.apk` 里的 so；
     * 需要看图标时再单独打开 dpi split。
   * 合并胖 APK 更多是为了方便自己装在不同设备上调试。

2. **生产环境/分发不建议这么做**

   * 合并后需要使用你自己的签名，无法覆盖安装原版；
   * 资源 ID 可能与原版不同，如果服务器端有奇怪校验，也可能踩坑。

3. **排查问题时，多关注 aapt2 报错信息**

   * XML 语法错误、资源重复、ID 冲突，几乎都能从 `apktool b` 的日志一眼看出来。

---

## 结语

这次对 X Saver 的 split APK 合并，主要经历了三个关键点：

1. 正确认识 base / ABI / DPI 三类 split；
2. **所有 XML 资源必须来自 apktool 解包后的工程，而不是 raw APK 解压出来的二进制 XML**；
3. Manifest 与 `public.xml` 的处理要克制：

   * Manifest 只删与 split 相关的 meta-data / 属性；
   * `public.xml` 保留 base 的，忽略 split 的。

掌握这一套之后，再遇到类似的 App Bundle 拆包场景，基本都可以按这个套路来处理、做成自己的胖 APK 版本，方便后续 Hook、日志、协议逆向等工作。
