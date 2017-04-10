# Retrofit
**Introduction:**
  A type-safe HTTP client for Android and Java

##  Import
MAVEN
```
<dependency>
<groupId>com.squareup.retrofit2</groupId>
<artifactId>retrofit</artifactId>
</dependency>
<version>2.2.0</version>
```
GRADLE
```
compile 'com.squareup.retrofit2:retrofit:2.2.0'
```
* need to add internet permission.
* Retrofit requires at minimum Java 7 or Android 2.3.

## Usage
  * Step1. define interface
  ```Java
  public interface GithubSerivice{
    @GET("users/{user}/repos");
    Call<List<repo>> listRepos(@Path("user") String user);
  }
  ```
  * Step2. generate a implemetion of the interface
  ```Java
  Retrofit retrofit = new Retrofit.Build()
  .baseUrl("https://api.github.com")
  .build();

  GithubService service = retrofit.create(GithubService.class);
  ```
  * Step3. make a synchronous or synchonous  HTTP request
  ```Java
  Call<list<Repo>> call = service.listRepos("Google");
  call.enqueue(new Callback<RequestBody>(){
    @Override
    public void onResponse(Call<ResponseBody> call,
    Response<ResponseBody> response) {
      try {
            String result = response.body().string();
            Log.i(TAG, result
          } catch (IOException e) {
            e.printStackTrace();
      }
    }

    @Override
    void onFailure(Call<ResponseBody> call, Throwable t) {

          }
  });
  ```

*** Annotations ***

* URL Prarameter replacement and queryparameter support
* Object converison to request body
* Multipart request body and file upload

## Add argument for a *** GET *** request

* @Query
  use `@Query("argName" String value)` add argument for get request.

* QueryMap
  `getXX (@QueryMap<String,String> params);``
  add one or more argument in a map.

* @Path
```Java
  @GET("api/data/Android/10/{page}")
  Call<GankBean> getAndroidInfo(@Path("page") int page);
  ```
  fill url by `@Path` Annotation

## use ***POST*** method

```Java
  public interface UserService(){

    @Post("user/login")
    Call<reSponseBody> login(@Body User user);
  }

```

##  commit ***Field***
```Java
  @POST(/user/edit)
  Call<ReSponseBody> editUser(@Field("id") int id,@Field("name") String name);
```
```Java
  call.editUser(1,"xiaoming")
  .enqueue(new CallBack<ResponseBody>(){
    //...
  });
```

## PUT method

## HEADER



## ***Retrofit + RxJava***

## Use a Interceptor from OkHttp
```Java
  //create a OkHttpClient Object
  OkHttpClient client = new OkHttpClient();
  client.interceptors().add(new Interceptor(){
    @Override
    public Response intercept(Chain chain ) thorows IOException {
        //do anything with response here
        return Response;
    }
  });

  //add client to Retrofit Builder
  Retrofit retrofit = new Retrofit.Builder()
  .baseUrl("xxx.com/")
  .addconvertFactory(GsonConverterFactory.create())
  .client(client)
  .build();
```

## Rxjava intergration with CallAdapter
