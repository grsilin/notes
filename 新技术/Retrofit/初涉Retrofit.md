## REST

REST（Representaional State Transfer）是一组架构约束条件和原则。

RESTful架构都满足以下条件：

1. 每一个URL代码一种资源
2. 客户端和服务器之间，传递这种资源的某种表现
3. 客户端通过四个HTTP动作，对服务器资源进行操作，实现“表现层状态转化”



## Retrofit2.0使用配置

引入依赖

```
    compile 'com.squareup.retrofit2:retrofit:2.0.2'
    compile 'com.squareup.retrofit2:converter-gson:2.0.2'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.0.2'

    compile 'com.squareup.okhttp3:logging-interceptor:3.2.0'
```

OkHttp配置

```java
HttpLoggingInterceptor interceptor =
new  HttpLoggingInterceptor();
        interceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
        client = new OkHttpClient.Builder()
                .addInterceptor(interceptor)
                .retryOnConnectionFailure(true)
                .connectTimeout(15, TimeUnit.SECONDS)
                .addNetworkInterceptor(authorizationInterceptor)
                .build();
```

其中Level有：BASIC / HEADERS / BODY

retryOnConnectionFailure:错误重联

addInterceptor:设置应用拦截器，可用于设置公共参数，头信息，日志拦截等

addNetworkInterceptor：网络拦截器，可以用于重试或重写

Retrofit配置

```java
        Retrofit retrofit = new Retrofit.Builder()
        .baseUrl(BASE_URL)
        .client(client)
        .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
        //提供RxJava支持
        .addConverterFactory(GsonConverterFactory.create(gson))
        .build();//提供Gson支持
        apiService = retrofit.create(ApiService.class);
```

## Retrofit2.0 使用介绍

```java
//定以接口
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}

//获取实例
Retrofit retrofit = new Retrofit.Builder()
    //设置OKHttpClient,如果不设置会提供一个默认的
    .client(new OkHttpClient())
    //设置baseUrl
    .baseUrl("https://api.github.com/")
    //添加Gson转换器
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);

//同步请求
//https://api.github.com/users/octocat/repos
Call<List<Repo>> call = service.listRepos("octocat");
try {
     Response<List<Repo>> repos  = call.execute();
} catch (IOException e) {
     e.printStackTrace();
}

//call只能调用一次。否则会抛 IllegalStateException
Call<List<Repo>> clone = call.clone();

//异步请求
clone.enqueue(new Callback<List<Repo>>() {
        @Override
        public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
            // Get result bean from response.body()
            List<Repo> repos = response.body();
            // Get header item from response
            String links = response.headers().get("Link");
            /**
              * 不同于retrofit1 可以同时操作序列化数据javabean和header
              */
        }

        @Override
        public void onFailure(Call<List<Repo>> call, Throwable t) {

        }
    });

// 取消
call.cancel();
```

```java
//rxjava support
public interface GitHubService {
  @GET("users/{user}/repos")
  Observable<List<Repo>> listRepos(@Path("user") String user);
}

// 获取实例
// Http request
Observable<List<Repo>> call = service.listRepos("octocat");
```

## Retrofit注解

方法注解：

@GET、@POST、@PUT、@DELETE、@PATH、@HEAD、@OPTIONS、@HTTP

标记注解，包含@FormUrlEncoded、@Multipart、@Streaming。
参数注解，包含@Query,@QueryMap、@Body、@Field，@FieldMap、@Part，@PartMap。

其他注解，@Path、@Header,@Headers、@Url

特殊的注解：

@HTTP:可以替代其他方法的任意一种

```JAVA
 /**
     * method 表示请的方法，不区分大小写
     * path表示路径
     * hasBody表示是否有请求体
     */
    @HTTP(method = "get", path = "users/{user}", hasBody = false)
    Call<ResponseBody> getFirstBlog(@Path("user") String user);
```

@Url:使用全路径复写baseUrl，适用于非统一baseurl的场景

```java
@GET
Call<ResponseBody> v3(@Url String url);
```

@Streaming:用于下载大文件

```java
@String
@GET
Call<ResponseBody> downloadFileWithDynamicUrlAsync (@Url String fileUrl);
```

```java
ResponseBody body = response.body();
long fileSize = body.contentLength();
InputStream inputStream = body.byteStream();
```

常用注解：

@Path：URL占位符，用于替换和动态更新,相应的参数必须使用相同的字符串被@Path进行注释