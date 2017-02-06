## OKHttp

### (1) 导入

> compile 'com.squareup.okhttp:okhttp:2.4.0'
>
> compile 'com.squareup.okio:okio:1.5.0'

注：OKHttp内部依赖okio

### (2)使用

**get请求**

```java
OKHttpClientManager.getAsyn(url,new OKHttpManager.ResultCallback<String>{
     @Override
            public void onError(Request request, Exception e)
            {
                e.printStackTrace();
            }

            @Override
            public void onResponse(String response)
            {
                mTv.setText(u);//注意这里是UI线程
            }
});
```

**上传文件且携带参数**

```java
OKHttpClientManager.postAsyn(url,new OKHttpClientManager.ResultCallback<String>{
          @Override
        public void onError(Request request, IOException e)
        {
            e.printStackTrace();
        }

        @Override
        public void onResponse(String result)
        {

        }
	},file,"name",//对应<input type="file" name="File" >
	new OKHttpClientManager.Param[]{
      new OKHttpClientManager.Param("arg1",arg1value),
       new OKHttpClientManager.Param("art2",arg2value)
	}
);
```

**文件下载**

```java
OKHttpClientManager.downloadAsyn(url, Environment.getExternalStorageDirectory().getAbsolutePath(), //dir
new OKHttpManager.ResultCallback<String>(){
    @Override
        public void onError(Request request, IOException e)
        {

        }

        @Override
        public void onResponse(String response)
        {
            //文件下载成功，这里回调的reponse为文件的absolutePath
        }
});
```

**展示图片**

```java
OKHttpClientManager.displayImage(mImageView,url);
```

**整合gson**

```java
OKHttpClientManager.getAsyn(url,new OKHttpClientManager.ResultCallback<User>(){
  @override
  public void onError(Request request , Exception e){
    e.printStackTrace();
  }
  @Override
  public void onResponse(User user){
    mTv.setText(user.toString());
  }
});
```

