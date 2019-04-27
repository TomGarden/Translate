## 0x00、 [android.support.v4.content.FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)

FileProvider 是 ContentProvider 的一个特殊子类，通过将 app 创建 `content://` Uri 的行为替换创建 `file://` Uri 的行为来文件共享相关操作。

一个 conteent URI 允许你授予临时的 读/写 权限。
当你创建包含 Uri 的 Intent 的时候，要发送 URI 的内容到三方应用，你可以调用 Intent.setFlags() 增加权限。
这个权限在 接收此 Intent 的 Activity 存活期间是有效的。
对于 Service 接收 Intent 的 Service running 期间权限是有效的。

相比之下，要控制 Uri(`file:///`) 的权限，你必须修改底层文件的权限。
这种权限对于任何应用都是可用的，知道你修改了这些权限。
这种级别的访问从根本上来说是不安全的。

由 FileProvider 操作的 content URI 所增加的安全访问功能是 Android 安全的基础设施。

FileProvider 相关的话题：
- Defining a FileProvider
- Specifying Available Files
- Retrieving the Content URI for a File
- Granting Temporary Permissions to a URI
- Serving a Content URI to Another App

### 0.1、 定义 一个 FileProvider

FileProvider 提供基础功能(根据 file 生成 content URI),你不需要定义子类。
就是说你可以在你的 app 中 在 xml 文件中定义 FileProvider 实体。
要指定 FileProvider 组件，你可以增加一个 `<provider>` 元素到你的 manifest 文件。
设置 `android:name="android.support.v4.content.FileProvider"` 。
设置 `android:authorities` 需要根据一个受你控制的域名，(如果你的域名为 mydomain.com),你应该设置  `com.mydomain.fileprovider`。
设置 `android:exported` 为 `false` ； FileProvider 无需公开。
设置 `android:grantUriPermissions="true"` ，这允许你将文件的临时访问权限授予给三方应用。

```xml
<manifest>
    ...
    <application>
        ...
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.mydomain.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            ...
        </provider>
        ...
    </application>
</manifest>
```

如果你想覆写 FileProvider 的任何函数，继承 FileProvider 类，之后 `android:name` 属性需要设置你自己的类的完全限定名称。

### 0.2、 指定可用的文件。

一个 FileProvider 只向将你曾经指定的文件夹中的文件授予权限。
所以你需要指定一个文件夹，将文件夹的路径定义在 XML 文件中(使用 `<paths>` 元素的子元素)。
例如，下面的 `paths` 元素告诉 FileProvider 你打算请求 你提供的路径下面的子文件夹 `images/` 的 content URI 。

```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path name="my_images" path="images/"/>
    ...
</paths>
```

`<paths>` 元素必须包含下面的一个或多个子元素 ：

-   `files/` 代表你 app 内部存储的子文件夹下的文件们。
    这个子文件夹的字段值和 `Context.getFilesDi()` 的值相等。
    ```xml
    <files-path name="name" path="path" />
    ```

-   下面一句代码代表你 app 内部存储路径中的 cache 子文件夹下的文件们。
    这句代码所指定的文件们的根路径的值与 `getCacheDir()` 的值相同。
    ```xml
    <cache-path name="name" path="path" />
    ```

-   下面代码表示的文件们的根是 外部存储 路径。
    根路径的值与 `Environment.getExternalStorageDirectory()` 值相同。
    ```xml
    <external-path name="name" path="path" />
    ```

-   下面代码表示的文件们的根路径是 app 外部媒体文件的路径。
    它所指定的子文件夹的根路径与 `Context.getExternalMediaDirs()` 的值相同。
    ```xml
    <external-media-path name="name" path="path" />
    ```
    - **注意** 这个路径只有 API 21+ 才可用

这些子元素全部使用了相同的属性。

-   `name="name"`  
    表示一个 URI 路径字段。
    为了安全，这个值隐藏了你打算分享的子文件夹的名字。
    子文件夹的名字包含在 `path` 属性中。

-   `path="path"`
    你打算文件的子文件夹。
    当 `name` 属性是一个 URI 路径字段的时候， `path` 的值是一个实际的子文件夹的名字。
    注意，这个 value 指向一个 **子文件夹** ，不是一个确定的文件或者一些文件。
    你不能通过文件名字分享单一文件，你也不能使用通配符指定子集。

你必须为你希望分享的文件夹在 `<paths>` 元素下指定子元素，这个子元素中包含了你希望包含的 URI。
下面的代码指定了两个文件夹 ：

```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path name="my_images" path="images/"/>
    <files-path name="my_docs" path="docs/"/>
</paths>
```

将这个 `<paths>` 元素和它的子元素添加到你项目中的 XML 文件中。
例如，你能增加一个新的文件 `res/xml/file_paths.xml` 。
将这个文件连接到 FileProvider ，增加一个 `<meta-data>` 元素作为 `<provider>` 元素的子元素， `<meta-data>`  元素的属性应该设置为 `android.support.FILE_PROVIDER_PATHS` 。
这只元素的 `android:resource` 属性为 `@xml/file_paths`(注意，你无需填写 `.xml` 后缀)。
例如：

```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.mydomain.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

### 0.3、 针对 File 生成 URI

要使用 URI 向其他 应用分享 文件，你的 app 必须生成 content URI。
要生成 Content URI ，你需要创建一个 `File` 对象，然后传送给 `getUriForFile()` 方法即可完成。
你能将 `getUriForFile()` 生成的(`return`) content URI 通过 Intent 发送给其他 app 。
客户端 app 接收到这个 content URI 可以通过 `ContentResolver.openFileDescriptor` 获取一个 `ParcelFileDescriptor` 从而可以访问这个文件。

例如，假设你的 app 正在通过 `FileProvider` 向三方 app 提供文件,假设它的 `authority` 为 `com.mydomain.fileprovider` 。
要获取 URI 指定的 `images/` 路径下的文件 `default_image.jpg` 可以使用：

```java
File imagePath = new File(Context.getFilesDir(), "images");
File newFile = new File(imagePath, "default_image.jpg");
Uri contentUri = getUriForFile(getContext(), "com.mydomain.fileprovider", newFile);
```

这个代码片段中： `getUriForFile()` `return` 的 content URI 为 ：  `content://com.mydomain.fileprovider/my_images/default_image.jpg` 。


### 0.4、 为 URI 授予临时权限

要给从 `getUriForFile()` 返回的 content URI 授予访问权限需要这么做：
-   调用 `Context.grantUriPermission(package, Uri, mode_flags)` 使用 期望的 权限 flag。
    这将授予 content URI 对指定包的临时访问权限，这个 flag 的值你可以设置 `FLAG_GRANT_READ_URI_PERMISSION` 或 `FLAG_GRANT_WRITE_URI_PERMISSION` 也可以这两个值一起设置。
-   这个权限会一直被持有，直到你通过 `revokeUriPermission()` 撤销授权，或者 设备重启。

-   通过 `setData()` 方法将 URI 打包到 `Intent` 中。
-   接下来调用 `Intent.setFlags()` 设置 `FLAG_GRANT_READ_URI_PERMISSION` 或者 `FLAG_GRANT_WRITE_URI_PERMISSION` 或者两个都设置。
-   最终将 Intent 发送给其他 app 。
    权限通过一个 intent 授予的话如果接收此 Intent 的 组件(Activity/Services )处于存活状态的话那么权限的影响力就一直存在。
    当接收 Intent 的组件出栈(执行结束)，权限被自动移除。
    被授予权限的客户端的某个 Activity 的权限会自动扩展到该客户端的其他组件。

### 0.5、 向其他应用程序提供 Content URI

针对一个 file 有多种方式将其 content URI 分享到其他 应用。
一个常见的方式是三方 app 通过 `startActivityForResult()` 调用你的 app ， 它发送一个 Intent 给你 app 中被启动的 Activity 。
作为结果，你的 app 能直接立刻返回一个 content URI 也可以提供一个接口供其选择一个文件在做返回。
在后一种情况下，一旦用户选择了文件，你的 app 就返回它的 content URI 。
这两种方式中，你的 app 都将 URI 通过 `setResult()` 放在　Intent 中　。

你也可以将 URI 放到一个 ClipData 对象中，然后将 ClipData 放到　Intent 中。
要做到这些，可以调用 `Intent.setClipData()` 。
当你使用这个方法的时候，你能增加多个 ClipData 对象到 Intent 中，每一个都携带了单独的 content URI。
当你调用 `Intent.setFlags()` 的时候，会将权限全部临时授予这些 URI 。

> **注意** `Intent.setClipData()` 方法只在 api 16+ 可用。 
    如果你希望兼容之前的版本，你应该发送一个含有 Content URI 的 Intent 。
    通过设置 `ACTION_SEND` 并且通过 `setData()` 设置 URI 。