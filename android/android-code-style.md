# Android 开发规范(参考阿里规范)

## 前言

一份合格的代码不应只满足于实现功能, 更应该遵循良好的规范. 遵循良好的代码规范有利于:

- 提升程序稳定性, 减少代码隐患, 降低故障率;
- 增强可扩展性, 大幅提高维护效率;
- 统一标准, 提升多人协作效率;
- 方便新人快速上手, 在项目组人员发生变动时保证项目进度;

这里梳理一下Android开发过程中需要注意的一些地方, 包括多个部分, 另外根据约束力强弱分为两类:

- **强制**: 如果不遵守会导致代码严重混乱, 后期维护复杂, 甚至会出现严重bug;
- **推荐**: 如果不遵守可能会导致代码描述不清, 理解困难, 导致功能越多维护越难的问题;
  
下面是规范正文

## 系统架构相关

这一部分是项目总体上的要求, 包括 系统设计, 命名方式, 可见性, 注释, 代码风格 等几部分.

### 系统设计

#### 强制

1. 不允许出现两段相同的逻辑块, 必须抽出为公共方法, 差异性使用参数控制, 避免修改时多处修改导致遗漏;
2. 不允许出现两段相同的处于同一逻辑组的复杂布局, 必须抽为单独的include/merge;
3. 不允许父类中出现子类具体方法, 如果需要的话可以父类定义抽象方法, 交由子类实现;
4. 不允许Activity内多Fragment之间的直接沟通, 必须通过Activity中转;

#### 推荐

1. 推荐使用MVP或者MVVM架构;
2. 推荐使用Kotlin语言;
3. 采用模块分类方式替代文件类别方式, 方便快速查找模块相关内容, 例: LoginActivity/LoginPreenter/LoginHttpRequest/LoginBean/LoginAdapter等所属同一登录模块的文件放入一个文件夹, 而不是所有activity放入一个文件夹, 所有adapter放入一个文件夹.

### 命名方式

#### 强制

1. 不允许出现中文命名方式;
2. java/kotlin文件使用大驼峰方式, 例: LoginActivity.kt, NewsAdapter.kt, NewsBean.java;
3. layout/drawable/anim/style等resource文件使用小写+下划线的方式, 例: login_activity.xml, login_logo.png;
4. 类定义使用大驼峰方式, 例: class LoginPresenter {}, class NewsBean {};
5. 对象使用小驼峰方式, 例: LoginPresenter loginPresenter, NewsBean newsBean;
6. 静态常量使用全大写+下划线的方式, 例: public static final boolean IS_RELESAE = true;
7. Kotlin使用的布局中的控件id必须使用小驼峰方式, 例: android:id="@+id/tvLogin"

#### 推荐

1. 文件/资源命名时采用 模块+类型 的方式, 以便迅速查找相关内容, 例如登录页面: LoginActivity.kt, login_activity.xml, login_logo.png, <string name="login_net_error">网络错误</string>, <color name="login_page_backgroud">#f3f3f3</color>

2. java使用的布局中的id名建议使用小驼峰方式, 并且使用控件类型缩写开头, 例: android:id="@+id/tvLogin", 附录常用控件缩写:

| 控件	| 缩写
|  ----  | ----  |
| LinearLayout | ll
| RelativeLayout | rl
| ConstraintLayout | cl
| ListView | lv
| ScollView | sv
| TextView | tv
| Button | btn
| ImageView | iv
| CheckBox | cb
| RadioButton | rb
| EditText | et

### 可见性

#### 强制

1. 所有新定义的类/方法, 默认写成private, 只有在其他类需要引用时再看情况标为public, protected, package-private;

#### 推荐

1. java定义的父类中定义的方法如果子类重写会导致问题时, 添加final关键字;

### 注释相关

#### 强制

1. 类/复杂或者不能从方法名字看出意图的方法必须添加注释, 当类/方法添加注释时, 必须使用此类型注释:

```java
/**
* Created by XXX on 2019/6/19.
* 描述此类作用, 逻辑复杂的说明一下主要思路
*/
public class LoginPresenter {
    /**
    * 用于进行网络请求
    * @params xxx XXX
    */
    public void doLoginRequest(...){}
}
```

2. 变量注释不允许使用与类/方法一致的注释形式;
3. 方法注释中不允许出现@params, @return的参数描述错误的情况, 必须实时更新;

#### 推荐

1. 一段逻辑建议使用/* */的方式;
2. 方法/参数建议添加 @Nullable, @NotNull, @UiThread 等注解;

### 代码风格

此git目录下同时存有 AndroidCodeStyleSetting.jar 配置文件, 用于AndroidStudio导入后按照统一风格进行代码的格式化.

如果没有编写代码时随时格式化代码的习惯, 可以在AndroidStudio版本控制提交窗口右侧Before Commit中勾选Reformat code选项.

## Android具体模块相关

主要包括 基本组件, UI/布局, 进程/线程/消息推送, 文件/数据库, 图片/动画, 安全性等几个部分.

### Android基本组件

#### 强制

1. Intent通信时不允许传递超过1M的数据, 可以采用外部Presenter中转或者EventBus传递的方式;
2. Intent隐式启动时必须检查目标是否存在, 否则会出现目标未找到崩溃: if (getPackageManager().resolveActivity(intent, PackageManager.MATCH_DEFAULT_ ONLY) != null);
3. Activity/Service/BroadcastReceiver内如果有耗时操作, 必须采用多线程进行处理;
4. 应用内部发送广播时, 只能使用LocalBroadcastManager.getInstance(this).sendBroadcast(intent), 不允许 context.sendBroadcast(intent), 避免外部应用拦截;
5. 不允许在Application中缓存数据, 全局的共享数据可以使用某presenter存储, 或者使用SharedPreference读写;
6. Activity或者Fragment中动态注册BroadCastReceiver时，registerReceiver 和unregisterReceiver必须要成对出现;

#### 推荐

1. Activity#onPause/onStop中结合isFinishing的判断来执行资源的释放, 必免放在执行时机较晚的Activity#onDestroy()中执行;
2. 不要在Activity#onPause中执行耗时操作, 这样会导致界面跳转卡顿, 可以放入Activity#onStop中执行;

### UI/布局

#### 强制

1. 布局xml优先使用ConstraintLayout, 可以保证无嵌套的情况下完成包括部分控件同时显隐需求在内的99%的布局要求;
2. 不允许使用ScrollView包裹ListView/GridView/ExpandableListVIew等列表View, 复杂多项式列表可以使用多ItemType进行处理;

#### 推荐

1. 在Activity中显示对话框或弹出浮层时, 尽量使用DialogFragment, 而非Dialog/AlertDialog, 便于随Activity生命周期管理弹窗的生命周期;

### 进程/线程/消息推送

#### 强制

1. 存在多进程的情况时, Application中的初始化代码要根据进程分别处理, 避免初始化不必要的业务;
2. 新建线程时, 必须通过线程池的方式, 不允许采用new Thread()的方式;
3. Activity/Fragment中使用Handler时, 必须使用静态内部类+WeakReferences方式或者在onStop中调用handler.removeCallbacksAndMessages;

#### 推荐

1. 多进程间共享数据使用ContentProvider替代SharedPreferences#MODE_MULTI_PROCESS;

### 文件/数据库

#### 强制

1. 使用系统API获取文件路径, 避免手拼字符串, 例: android.os.Environment#getExternalStorageDirectory(), Context#getFilesDir(), 错误示例: File file = new File("/mnt/sdcard/Download/Album", name);
2. 当使用外部存储时, 必须检查外部存储的可用性: Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState());
3. 数据库Cursor使用之后必须关闭, 以免内存泄漏;

#### 推荐

1. SharedPreference仅存储简单数据类型, 不要存储复杂数据, 如json数据/Bitmap编码等;
2. SharedPreference提交数据时, 尽量使用Editor#apply(), 而非Editor#commit();

### 图片/动画

#### 强制

1. 加载大图时必须在子线程中处理, 否则会卡UI;
2. 在Activity.onPause()/onStop()中关闭当前activity正在执行的动画;

#### 推荐

1. Android图片建议转化为WebP格式, 可以减少APK体积;
2. 动画尽量不要使用AnimationDrawable, 占用非常多内存;
3. 使用ARGB_565代替ARGB_888, 减少内存占用;
4. 当Animation执行结束时, 调用View.clearAnimation()释放相关资源;

### 安全性

####强制

1. 上线包必须混淆;
2. 加解密的秘钥/盐不允许硬编码到代码中, 以防反编译获取;
3. Https处理时必须校验证书, 不允许直接接受任意证书;
4. 使用Android的AES/DES/DESede加密算法时, 不要使用默认的加密模式ECB, 应显示指定使用CBC/CFB加密模式;
5. 禁止把敏感信息打印到 log 中;
6. 在应用发布时必须确保android:debuggable为false;
7. 必须利用X509TrustManager子类中的checkServerTrusted函数效验服务器端证书的合法性,
8. 必须将android:allowbackup属性设置为false, 防止adb backup导出应用数据;
