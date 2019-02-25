## 0x00、 [Retrofit](https://square.github.io/retrofit/)

[原文](https://square.github.io/retrofit/)

应用于 Android 和 Java 品台的类型安全的 Http 客户端。

## 0x01、 简介

Retrofit 将你的 HTTP API 转换为 Java 接口。

```java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

用 `Retrofit` 实现 `GitHubService` 接口。

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

每一次调用 `GitHubService.call` 都能制造一个同步/异步 Http 请求。

```java
Call<List<Repo>> repos = service.listRepos("octocat");
```

使用注解描述 HTTP 请求：
- URL 参数替换和查询参数支持
- 对象转换为请求体 (例如:JSON 协议缓存)
- 多部分请求体和文件上传

## 0x02、 API 声明

接口方法的注解以及参数表明了请求将如何构建。

### 2.1、 请求方法

每一个方法必须有一个 HTTP 注解，其中包含了请求方法和相对 URL 。
 5 个基本请求方法： GET，POST，PUT，DELETE，HEAD 。
资源的相对 URL 在注解中指定。

```java
@GET("users/list")
```

你也能在 URL 中指定查询参数：

```java
@GET("users/list?sort=desc")
```

### 2.2、 URL 操作

请求 URL 能使用更新块在方法内部动态更新。
替换块是被花括号包含的字符串 `{balabala}`。
与替块相对应的参数必须被 `@Path` 修饰，并且注解中的字段与替换块总的字段相同。

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```

也可以添加请求参数。

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
```

对于复杂的请求参数可以使用 `Map` 进行组合。

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```

### 2.3、 请求体

一个对象能被指定为请求体(通过 `@Body` 注解)。

```java
@POST("users/new")
Call<User> createUser(@Body User user);
```

这个对象也将使用 `Retrofit` 实例上指定的转换器进行转换。
如果没有添加转换器，就只能使用 `RequestBody` 。

### 2.4、 编码和多部分

方法也能被声明去发送被编码的数据以及多部分数据。

当为方法添加 `@FromUrlEncoded` 注解的时候，可以用来发送编码数据。
每一个 键值 都由 `@Field` 提供键名对象置空值对象。

```java
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```

`@Multipart` 注解修饰的方法可以分部分请求。
使用 `@Part` 声明每一部分。

```java
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```

多部分使用 `Retrofit` 的转换，或者能实现 `RequestBody` 持有自己的序列化。

### 2.5、 操作 请求头(header)

你能通过为方法添加 `@Headers` 注解设置请求头。

```java
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
```

```java
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```

注意请求头不会相互覆盖。
所有同名的请求头都会被包含到请求中。

请求头能使用 `@Header` 动态更新。
相应的参数必须提供 `@Header` 注解。
如果值是 `null` ，请求头将被省略。
另外， `toString` 将被调用返回值作为结果被使用。

```java
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```

类似请求参数，对于复杂的请求头，可以使用 `Map` 。

```java
@GET("user")
Call<User> getUser(@HeaderMap Map<String, String> headers)
```

可以使用 [OkHttp拦截器](https://github.com/square/okhttp/wiki/Interceptors) 指定需要添加到每个请求的 请求头。

### 2.6、 同步和异步

`Call` 实例能同步(或异步)执行。
每一个实例只能被调用一次，但是调用 `clone()` 会创建新的可用的实例。

在 Android 中， 回调将会在 主线程中 被执行。
在 JVM ，回调将会在调用 HTTP 请求的线程中被执行。

## 0.03、 Retrofit 配置

`Retrofit` 是将你的 API 接口转换为可调用的对象的 类。
默认的， Retrofit 将使用最适合你平台的做法，但是这是可定制的。

### 3.1、 转换

默认的， Retrofit 只能反序列化 HTTP body 到 OkHttp 的 RequestBody 类型，并且它只能接收 `ReqeustBody` 类型(需要 `@Body` 注解)。

可以添加转换器以支持其他类型。 六个兄弟模块适应流行的序列化库，方便您使用。

- Gson: com.squareup.retrofit2:converter-gson
- Jackson: com.squareup.retrofit2:converter-jackson
- Moshi: com.squareup.retrofit2:converter-moshi
- Protobuf: com.squareup.retrofit2:converter-protobuf
- Wire: com.squareup.retrofit2:converter-wire
- Simple XML: com.squareup.retrofit2:converter-simplexml

这里有一个使用 `GsonConverterFactory` 生成 `GitHubService` 接口实现类的例子(使用 Gson 作为反序列化)。

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

### 3.2、 自定义转换器

如果您需要与使用Retrofit不支持开箱即用的内容格式的API（例如YAML，txt，自定义格式）进行通信，或者您希望使用其他库来实现现有格式，则可以轻松创建 你自己的转换器。 创建一个 `Converter.Factory` 的扩展类，并在构建适配器时传入实例。

