---
layout:     post
title:      "用Retrofit2上传文件"
subtitle:   "Alamofire+ObjectMapper+RealmSwift"
date:       2016-08-24 09:42:00
author:     "ZoeIce"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - android
    - Retrofit2
---

>如何用Retrofit2上传图片文件

使用Retrofit2, 你需要将OkHttp的`RequestBody`或者`MultipartBody.Part`中的一个或多个作为请求body

你需要定义一个服务接口，代码如下：

```java
public interface FileUploadApi {
    //文件上传
    @Multipart
    @POST("media/upload")
    Call<HttpResult> uploadFile(
            @Part("desc") RequestBody description,
            @Part MultipartBody.Part file
    );
}
```

用`ServiceGenerator`来创建一个服务对象，

代码如下：

```java
public class FileUploadClient {
    private static FileUploadClient instance = new FileUploadClient();

    public static final String MULTIPART_FORM_DATA = "multipart/form-data";

    private FileUploadClient() {
    }

    public static FileUploadClient Instance() {
        return instance;
    }

    private static OkHttpClient.Builder httpClient = new OkHttpClient.Builder();

    private static Retrofit.Builder builder =
            new Retrofit.Builder()
                    .baseUrl("http://xxx.com/xxx")
                    .addConverterFactory(GsonConverterFactory.create());

    public static <S> S createService(Class<S> serviceClass) {
        Retrofit retrofit = builder.client(httpClient.build())
                .build();
        return retrofit.create(serviceClass);
    }

    public <T> void updateFile(String filePath, final Class<T> clazz, @NonNull final ApiCallback<T> apiCallback){
        FileUploadApi service = createService(FileUploadApi.class);

        File file = new File(filePath);

        // 创建文件想关的 RequestBody
        RequestBody requestFile =
                RequestBody.create(MediaType.parse(MULTIPART_FORM_DATA), file);

        // MultipartBody.Part 也用于实际发送的文件名
        MultipartBody.Part body =
                MultipartBody.Part.createFormData("picture", file.getName(), requestFile);

        // 增加描述信息
        String descString = "descString";
        RequestBody desc =
                RequestBody.create(MediaType.parse(MULTIPART_FORM_DATA), descString);

        // 执行上传请求
        Call<HttpResult> call = service.uploadFile(desc, body);
        call.enqueue(new Callback<HttpResult>() {
            @Override
            public void onResponse(Call<HttpResult> call,
                                   retrofit2.Response<HttpResult> response) {
                if (response.body() != null) {
                    HttpResult result = response.body();
                    if (result.code == 0 || result.data != null) {
                        Gson gson = new Gson();
                        T data = gson.fromJson(result.data, clazz);
                        apiCallback.onResponse(data);
                    } else {
                        apiCallback.onFailure(result.errorMsg, result.code);
                    }
                } else {
                    apiCallback.onFailure(String.format("网络请求失败:%d", response.raw().code()), -1);
                }
            }

            @Override
            public void onFailure(Call<HttpResult> call, Throwable t) {
                apiCallback.onFailure(t.getMessage(), -1);
            }
        });
    }

}

```
