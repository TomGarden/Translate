## Android 研发 杨铭 · 5年 / 本科 / 简要

## 一、 个人信息

<table >
    <tr>
        <td>姓名</td><td>:</td><td>杨铭(♂)</td>
        <td>笔记</td><td>:</td><td><a href="https://github.com/TomGarden/tom-notes/issues">GitHub Issues Blog</a></td>
    </tr>
    <tr>
        <td>手机/微信</td><td>:</td><td>17611261931</td>
        <td>翻译 GFW</td><td>:</td><td><a href="https://tom.gitbook.io">tom.gitbook.io</a></td>
    </tr>
    <tr>
        <td>邮箱</td><td>:</td><td><a href="mailto:tom.work@foxmail.com">tom.work@foxmail.com</a></td>
        <td>学生时代CSDN</td><td>:</td><td><a href="https://blog.csdn.net/u014587769">blog.csdn.net/u014587769</a></td>
    </tr>
</table>

- 2014 ~ 至今，一直围绕 Android 成长 (不论是 Android 研发，Android 应用扫描，渗透测试)。 <br/>
  我的作品 : [Google Play 俄罗斯方块](https://play.google.com/store/apps/details?id=io.github.TomGarden.tetris)

## 二、 专业技能

### 2.1、 English
- 持续学习&练习 , 提高英文水平，[我的一点翻译 GFW](https://tom.gitbook.io)

### 2.2、 关于研发
- 架构模式 : MVC , VMP , MVVM [🔗](https://github.com/TomGarden/tom-notes/issues/178)
- 设计模式 : 观察者模式 、 装饰者模式 、 工厂模式 、 单例模式 。 
- 对安全、加固、逆向、自动化测试有一定的理解。
- Kotlin(2019~至今) , Java(2014~至今) , C++(2022~至今) ; 熟练使用 Kotlin 进行项目开发

### 2.3、 Android
1. 熟悉四大组件 : Activity、ContentProvider、BroadCast、Service 。
2. 熟悉常用组件控件与UI布局 : Intent 、 Fragment 、 ConstraintLayout 、 Dialog。
3. 熟悉多线程编程和线程间的通信机制 Handler、 Message、 MessageQueue、 Lopper 。[🔗](https://github.com/TomGarden/tom-notes/issues/179)
4. 熟悉控件联动、自定义控件、事件分发、手势拦截与处理并多有实践。
5. 熟悉 SVG、属性动画、图片处理、2D绘图并对 3D 绘图曾有所应用。
6. 熟悉文件存储、XML、SQLite/ContentProvider、网络、文件。
7. 熟悉 HTTP 网络通信、Socket 通信。
8. 在项目中应用过蓝牙、网卡、摄像头……等硬件和传感器。
9. 熟悉屏幕适配以及 Android 主题模块的业务逻辑。
10. 具备独立开发能力，能独立分析并解决常见客户端崩溃和异常问题。
11. 熟练使用主流开发库 RxJava、Retrofit、OkHttp 。


## 三、 主要经历

### 3.1、 [北京嘀嘀无限科技发展有限公司](https://www.didiglobal.com/) 
- <img src="SRC/images/didi_logo.jpeg" width = "20" height = "20" style="vertical-align:text-top" /> `2020.11.12~2022.06.21`
- 效能平台部 Android 研发 , 内部通讯 app 待办模块 . 
- 橙心优选 , 货物清点进销存 Android 客户端 . 

### 3.2、 [北京蜂群科技有限公司](https://manbanapp.com/)
- <img src="SRC/images/manban_fengqunkeji_logo.png" width = "16" height = "16" style="vertical-align:text-top" /> `2019.03.07~2020.11` 
- 独立开发、维护 「满班APP」 Android 客户端 ; 
    [下载了解](https://manbanapp.cn/)
- 创业公司、教育行业、教务管理系统、sass


### 3.3、 `2013.9.1~2017.6.30` 河北北方学院

- 计算机科学与技术专业　统招本科　二批次 
- Android 研发 [河北省人口健康信息化工程技术研究中心·移动互联研究室](http://kyc.hebeinu.edu.cn/webPage/showarticle1024.html)


## 四、 项目经历 

### 4.1. D-Chat(类飞书通讯工具) - Android
- 开发维护 [待办] 模块
- 技术细节
   - 引入并使用 Kotlin 开发维护
   - 为了用户离线情况下也可以操作数据 , 使用了对象型数据库(Realm) ; 
   - 为了多端动画效果一致 , 使用了动画三方库 Lottie ; 
   - 组件化开发 , 涉及到的工具 : Gradle + AS 模块化插件

### 4.2. [满班](https://manbanapp.cn/) - Android
- 开发维护 Android 客户端
- 技术细节
   - 引入并使用 Kotlin 开发维护
   - Fragment + Activity 页面组装
   - 图片处理 : Glide 
   - 依赖注入 : Dagger
   - 网络 : RxJava、 Retrofit、 OkHttp、 Gson
   - 视频压缩 : [ffmpeg / MediaCodec](https://github.com/TomGarden/tom-notes/tree/master/_posts/Collections/Android/point4dev/短视频)
   - 图片压缩 
   - 类似微信朋友圈功能 : RecyclerView + RecyclerViewAdapterHelper 
   - 性能监控 : 友盟 / 阿里EMAS apm
   - 日历视图 : 基于 [Android-Week-View](https://github.com/alamkanak/Android-Week-View) 定制自定义需求
   - 数据库操作 : Room
   - 蓝牙小票打印机
   - 二维码扫描与生成
   - ...


### 4.3、 俄罗斯方块(Android)
- `2018.10~2018.12` - 独立开发 第四次重构
- 技术细节
  - 重构时使用 MVP 架构和面向接口编程的思想使项目的可维护性和扩展性得到很大提高。
  - 自定义控件： 游戏界面的自定义和绘制， 较市面上已有的游戏界面更加简约精美。
  - 多线程： 运行过程中的多线程管理。
  - 屏幕触控： 检测并响应滑动点击等手势操作。
  - 属性动画： 游戏细节动画，SVG 按钮动画。
  - 主题: 主题控制和定制。
- 项目地址
  - https://github.com/TomGarden/Tetris

### 4.4、 法律法规(Android)
- `2016.11-2017.1` - 独立开发（含服务端接口）
- 项目介绍
  - 工具类软件， 主要用来在医疗卫生行业快速查找、 查看、 下载法律法规条文。 本软件最终会集成到定制的 Android 手机并以手机的形式提供给医疗卫生行业的执法者。
- 技术细节
  - 自定义控件： 阅读界面的定制。 模仿翻页效果。
  - 数据本地化： SQLite 存储本地化的法律法规， XML 存储用户个性数据和配置信息。
  - HTTP 通信： OKHttp 和服务端完成 Json 通信获取法律法规目录或内容等。
  - 注解： 通过 Butter Knife 完成快捷开发。
  - 服务端接口的开发和对接： 主要是通过 hibernate 和 HQL 操作服务端数据库并对请求做出合理的响应。

### 4.5、 速传(Android+Windows)
- `2016.04-2016.06` - 团队开发（独立负责 Android 端）
- 项目介绍
  - 工具助手类软件。 主要通过二维码和雷达功能完成 Windows 和 Android 平台无障碍快速文件共享传输。
- 负责的技术细节：
  - 主要是通过 ContentResolver 和文件遍历的方式获取系统数据。 并把数据展示给用户。
  - 这里的难点在内容的流畅显示， 通过 AsyncTask、 线程池、 LruCache 的处理是界面尽可能的流畅。 防止 oom。 从而可以在展示大量高清图片的时候不卡顿。
  - 连接阶段用 Json 封装两端数据进行设备交流。
  - 通过搭建 Socket 信道使用 Stream 完成设备间快速通信。 从而完成文件共享。

### 4.6、 万能播放器(Android)
- `2015.03-2015.06` - 团队开发
- 项目介绍
  - 一款支持视频、 音频、 图片浏览和处理的全方位播放器软件。
- 我的职责
  - 视频模块的编码， 框架搭建， 协调团队共同完成项目开发。
- 负责的技术细节：
 - 通过深度优先遍历的非递归实现算法。 达到更快速的搜索手机文件的目的。
 - 通过使用 VitamioSDK 来播放本地视频。 支持绝大多数已知视频格式的播放。
 - 支持本地文件的删除， 修改等文件管理操作。 类似文件管理器。
 - 显示大量视频文件高清图片的显示有效避免 oom 。

### 4.7、 手机遥控(Android)
- `2014.11-2015.01` - 独立开发
- 项目描述
  - 手机和电脑在同一局域网内完成手机对电脑的操作。 Android 端通过部署在 Windows 客户端的软件达到控制电脑的目的。
- 技术细节
  - Socket 通信， 通过手机和电脑间的通信完成相互控制的目的。


## END [详细信息请跳转 🔗](https://github.com/TomGarden/Translate/blob/master/About/CV_desktop_details_2023-06-07.md)

