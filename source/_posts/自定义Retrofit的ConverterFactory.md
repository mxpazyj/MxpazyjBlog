---
title: 自定义Retrofit的ConverterFactory
date: 2016-07-05 10:42:52
tags: Android
---
在我们与后台的交互的情况下，通常会定义一个固定的实体类

以我这里的实体类为例
#### 返回实体类
```java
public class HttpResult<T> {
    private String msgId;
    private int resultCode;
    private String resultMsg;
    private String trackingCode;
    private T data;
}
```
这里的getter setter方法我省略掉了

这里我们看到，所有的data字段，我都是用泛型表示，这里才是我们扩展的地方，其他地方都是固定的，在UI界面，一般情况，只在乎请求的成功或者失败，UI只需要data的数据，其他数据一概跟UI没有关系，由这里提出需求，在返回数据给UI之前，我们就将其他数据处理完毕。

一般我们实例化Retrofit的时候都会这样实例化
```java
new Retrofit.Builder().client(clientBuilder.build())
                      .baseUrl(API)
                      .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                      .addConverterFactory(GsonConverterFactory.create())
                      .build();
```
这里的GsonConverterFactory是我们今天自定义的重点

修改GsonConverterFactory固定我们的json body的实体类型

```java
@Override
public Converter<ResponseBody, ?> responseBodyConverter(final Type type, Annotation[] annotations, Retrofit retrofit) {
    Type newType = new ParameterizedType() {
        @Override
        public Type[] getActualTypeArguments() {
            return new Type[] { type };
        }
        @Override
        public Type getOwnerType() {
            return null;
        }
        @Override
        public Type getRawType() {
            return HttpResult.class;
        }
    };
    TypeAdapter<?> adapter = gson.getAdapter(TypeToken.get(newType));
    //上面代码的作用是获得HttpResult的TypeAdapter
    return new GsonResponseBodyConverter<>(adapter);
}
```
`GsonResponseBodyConverter.java`
```java
final class GsonResponseBodyConverter<T> implements Converter<ResponseBody, Object> {
    private final TypeAdapter<T> adapter;

    GsonResponseBodyConverter(TypeAdapter<T> adapter) {
        this.adapter = adapter;
    }

    @Override public Object convert(ResponseBody value) throws IOException {
        try {
            HttpResult apiModel = (HttpResult) adapter.fromJson(value.charStream());
            //拿到HttpResult实例通过对返回码进行判断，非正常code则抛出自定义异常
            //在onError中捕获异常
            if (apiModel.getResultCode() == 0) {
                return apiModel.getData();
            }
        } finally {
            value.close();
        }
        return null;
    }
}
```
`RequestCodeException.java`
```java
public class RequestCodeException extends RuntimeException {
    private int errCode;
    public RequestCodeException(int errCode) {
        this.errCode = errCode;
    }

    public RequestCodeException(String detailMessage) {
        super(detailMessage);
    }

    @Override
    public String getMessage() {
        String mMessage;
        switch (errCode){
            case 1:
                mMessage =  "用户已失效";
                break;
            ...
            ...
            default:
                mMessage =  "其他错误";
                break;

        }
        return mMessage;
    }
}
```
大功告成

对比一下

之前的Retrofit请求

```java
Observable<HttpRequest<UserInfo>> updateServerInfo(@Body HttpRequest<ApplyServerInfo> httpRequest);
```
之前的调用
```java
.subscribe(new Subscriber<HttpRequest<UserInfo>>() {
    @Override
    public void onCompleted() {
    }
    @Override
    public void onError(Throwable e) {

    }
    @Override
    public void onNext(HttpRequest<UserInfo> httpRequest) {
      if(httpRequest.getResultCode != 0){
        //处理失败的请求
      }else {
        //处理成功
      }
    }
});
```
现在的Retrofit请求
```java
Observable<UserInfo> updateServerInfo(@Body HttpRequest<ApplyServerInfo> httpRequest);
```
现在的调用
```java
.subscribe(new Subscriber<UserInfo>() {
    @Override
    public void onCompleted() {
    }
    @Override
    public void onError(Throwable e) {
      String errMsg = e.getMessage();
      //处理失败的请求
    }
    @Override
    public void onNext(UserInfo userInfo) {
      //处理成功
    }
});
```
这就是这次的优化结果，将对ResultCode的判断封装到GsonResponseBodyConverter中，使onNext、onError里面的代码逻辑清晰。

如果项目中对接口请求返回有加密解密的需要的，原理也一样。

在GsonConverterFactory中的responseBodyConverter进行解密，在requestBodyConverter进行加密
