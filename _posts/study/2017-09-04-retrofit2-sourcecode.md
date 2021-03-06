---
layout:     post
title:      "Retrofit2源码分析"
subtitle:   "Read The Fucking Source Code"
date:       2017-09-04 12:00:00
author:     "wenhao"
catalog:    true
tags:
    - study
---
## Retrofit2
---
### Retrofit2基本用法

```java
public interface GitHubService {
    @GET("users/{user}/repos")
    Call<ResponseBody> listRepos(@Path("user") String user);
}

Retrofit retrofit = new Retrofit.Builder()
               .baseUrl("https://api.github.com/")
               .build();

       GitHubService service = retrofit.create(GitHubService.class);

       final retrofit2.Call<ResponseBody> repos = service.listRepos("hao0917");

       repos.enqueue(new Callback<ResponseBody>() {
           @Override
           public void onResponse(retrofit2.Call<ResponseBody> call, Response<ResponseBody> response) {
           }

           @Override
           public void onFailure(retrofit2.Call<ResponseBody> call, Throwable t) {
           }
       });

```

### Retrofit对象的初始化

首先使用建造模式进行Builder()初始化
```java
public Builder() {
  this(Platform.get());//Platform平台信息
}

Builder(Platform platform) {
      this.platform = platform;

      //添加默认的结果转化器
      converterFactories.add(new BuiltInConverters());
}
```
调用Platform.get()方法对Paltform进行初始化

```java

private static final Platform PLATFORM = findPlatform();

 static Platform get() {
   return PLATFORM;
 }

 private static Platform findPlatform() {
    try {
      Class.forName("android.os.Build");//通过反射判断是否是Android平台
      if (Build.VERSION.SDK_INT != 0) {
        return new Android();
      }
    } catch (ClassNotFoundException ignored) {
    }
    ******
  }

  static class Android extends Platform {
    @Override public Executor defaultCallbackExecutor() {
      return new MainThreadExecutor();
    }

    @Override CallAdapter.Factory defaultCallAdapterFactory(@Nullable Executor callbackExecutor) {
      if (callbackExecutor == null) throw new AssertionError();
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }

    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
  }
```
在Android中初始化了defaultCallbackExecutor（MainThreadExecutor）和defaultCallAdapterFactory（ExecutorCallAdapterFactory）
MainThreadExecutor的作用是执行最终结果回调，ExecutorCallAdapterFactory的作用是对请求Call的转化。

－－－

Builder()完成后，进行setBaseUrl（）等参数设置，然后调用build（）方法

```java
public Retrofit build() {
  if (baseUrl == null) {
    throw new IllegalStateException("Base URL required.");
  }

  okhttp3.Call.Factory callFactory = this.callFactory;
  if (callFactory == null) {
    callFactory = new OkHttpClient();   //使用OkHttpClient
  }

  Executor callbackExecutor = this.callbackExecutor;
  if (callbackExecutor == null) {
    callbackExecutor = platform.defaultCallbackExecutor();//platform的Android中的MainThreadExecutor
  }

  // Make a defensive copy of the adapters and add the default Call adapter.
  List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
  adapterFactories.add(platform.defaultCallAdapterFactorjy(callbackExecutor));//platform的Android中的ExecutorCallAdapterFactory

  // Make a defensive copy of the converters.
  List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);//有默认的转换器BuiltInConverters

  return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
      callbackExecutor, validateEagerly);
}
}
```

我们看到 在Retrofit中如果用户未设置网络请求库的话系统默认使用OkHttpClient，然后把在Platform中初始化的
defaultCallbackExecutor（MainThreadExecutor）作为callbackExecutor
把defaultCallAdapterFactory（ExecutorCallAdapterFactory）添加到adapterFactories集合中，
把默认的BuiltInConverters添加到converterFactories集合中，
adapterFactories集合主要是对Call进行转化，我们使用到Rxjava就是被添加至次集合
converterFactories集合是对返回结果进行转化，我们使用到Gson就是被添加至次集合
至此我们对Retrofit的初始化就结束了，获取了retrofit对象。


### retrofit.create(）
```java
public <T> T create(final Class<T> service) {
   Utils.validateServiceInterface(service);    //service只能为interface且不可继承
   if (validateEagerly) {                      //是否对所有方法loadServiceMethod （147行）
     eagerlyValidateMethods(service);
   }
   return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
       new InvocationHandler() {
         private final Platform platform = Platform.get();

         @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)//method调用的方法 args方法的参数
             throws Throwable {
           // If the method is a method from Object then defer to normal invocation.
           if (method.getDeclaringClass() == Object.class) {   // method是一个对象的method
             return method.invoke(this, args);
           }
           if (platform.isDefaultMethod(method)) {             //java8中
             return platform.invokeDefaultMethod(method, service, proxy, args);
           }
           ServiceMethod<Object, Object> serviceMethod =       //Android中
               (ServiceMethod<Object, Object>) loadServiceMethod(method);  //获取method中所有信息，注解，参数，返回等。
           OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
           return serviceMethod.callAdapter.adapt(okHttpCall); //ExecutorCallAdapterFactory中的adapt方法
         }
       });
 }


```
create()主要使用了动态代理，动态代理的作用是可以在你要调用某个Class的方法时，插入你想要执行的代码
invoke方法中methon参数也就是我们触发代理时调用的方法，args为方法的参数
例子中是我们调用service.listRepos（）时触发，因此methon就是listRepos，args就是“hao0917”
首先执行的
 ServiceMethod<Object, Object> serviceMethod =
    (ServiceMethod<Object, Object>) loadServiceMethod(method);


```java
ServiceMethod<?, ?> loadServiceMethod(Method method) {
  ServiceMethod<?, ?> result = serviceMethodCache.get(method);
  if (result != null) return result;

  synchronized (serviceMethodCache) {
    result = serviceMethodCache.get(method);
    if (result == null) {
      result = new ServiceMethod.Builder<>(this, method).build();
      serviceMethodCache.put(method, result);
    }
  }
  return result;
}
```
如果有缓存就获取缓存中对象，否则调用new ServiceMethod.Builder<>(this, method).build();创建
并存入缓存中。
首先是ServiceMethod.Builder<>(this, method),主要是通过反射获取method的相关属性
```java
Builder(Retrofit retrofit, Method method) {
  this.retrofit = retrofit;
  this.method = method;
  this.methodAnnotations = method.getAnnotations();         //获得所有注解
  this.parameterTypes = method.getGenericParameterTypes();  //获得参数类型
  this.parameterAnnotationsArray = method.getParameterAnnotations();//获得所有参数
}
```
然后是调用ServiceMethod.Builder的build()方法
```java
public ServiceMethod build() {              //该方法是对注解参数等的具体解析
  callAdapter = createCallAdapter();        //获取Call转换 ExecutorCallAdapterFactory.get()
  ******
```

在build()方法中首先是调用createCallAdapter()方法生成callAdapter
```java
private CallAdapter<T, R> createCallAdapter() {
   Type returnType = method.getGenericReturnType();  //获取返回类型
   if (Utils.hasUnresolvableType(returnType)) {    //对返回类型进行检查
     throw methodError(
         "Method return type must not include a type variable or wildcard: %s", returnType);
   }
   if (returnType == void.class) {                   //对返回类型进行检查
     throw methodError("Service methods cannot return void.");
   }
   Annotation[] annotations = method.getAnnotations();//获得所有注解
   try {
     //noinspection unchecked
     return (CallAdapter<T, R>) retrofit.callAdapter(returnType, annotations);
   } catch (RuntimeException e) { // Wide exception range because factories are user code.
     throw methodError(e, "Unable to create call adapter for %s", returnType);
   }
 }
```
createCallAdapter()又调用了retrofit对象的callAdapter方法

```java
public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
  return nextCallAdapter(null, returnType, annotations);
}

public CallAdapter<?, ?> nextCallAdapter(@Nullable CallAdapter.Factory skipPast, Type returnType,
    Annotation[] annotations) {
  checkNotNull(returnType, "returnType == null");
  checkNotNull(annotations, "annotations == null");

  int start = adapterFactories.indexOf(skipPast) + 1;
  for (int i = start, count = adapterFactories.size(); i < count; i++) {
    CallAdapter<?, ?> adapter = adapterFactories.get(i).get(returnType, annotations, this);
    if (adapter != null) {
      return adapter;
    }
  }
````
最后调用了adapterFactories集合中的get方法，也就是platform.defaultCallAdapterFactory,即ExecutorCallAdapterFactory
```java

  @Override
  public CallAdapter<?, ?> get(Type returnType, Annotation[] annotations, Retrofit retrofit) {
    if (getRawType(returnType) != Call.class) {
      return null;
    }
    final Type responseType = Utils.getCallResponseType(returnType);
    return new CallAdapter<Object, Call<?>>() {
      @Override public Type responseType() {
        return responseType;
      }

      @Override public Call<Object> adapt(Call<Object> call) {    //传进来的是OkHttpCall
        return new ExecutorCallbackCall<>(callbackExecutor, call);
      }
    };
  }
```
返回的是CallAdapter 。因此build（）方法中调用createCallAdapter()最终返回的是CallAdapter。同理
调用createResponseConverter()返回converter。
至此loadServiceMethod（）走完，声称了ServiceMethod对象，该对象主要是获取网络请求接口 参数 返回等信息，以及对请求过程和返回过程处理方式。
然后在invoke方法中调用了
OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
生成了OkHttpCall对象,该对象是对OKHttp的Call的封装。
然后在invoke方法中调用了
serviceMethod.callAdapter.adapt(okHttpCall);
也就是ExecutorCallAdapterFactory中调用get生成的CallAdapter对象的adapt方法
```java
@Override
public Call<Object> adapt(Call<Object> call) {    //传进来的是OkHttpCall
    return new ExecutorCallbackCall<>(callbackExecutor, call);
}
```
生成了ExecutorCallbackCall
至此invoke（）方法走完，动态代理走完，retrofit.create(GitHubService.class);方法走完
调用 final retrofit2.Call<ResponseBody> repos = service.listRepos("hao0917");
返回retrofit2.Call 也就是ExecutorCallbackCall对象
### Call.enqueue

最终我们调用了Call的enqueue方法，也就是ExecutorCallbackCall对象的enqueue方法
```java
static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;  //回调线程池 默认platform中的MainThreadExecutor
    final Call<T> delegate;           //OkHttpCall

    ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
    }

    @Override public void enqueue(final Callback<T> callback) {
      checkNotNull(callback, "callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }

        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
    }
```
调用了OkHttpCall。在该类中把ServiceMethod中获取的参数 注解等转化为Request对象
并生成OkHttp 的 Call。
```java
private okhttp3.Call createRawCall() throws IOException {
    Request request = serviceMethod.toRequest(args);  //通过ServiceMethod中的参数创建OKHttp的Request对象
    okhttp3.Call call = serviceMethod.callFactory.newCall(request);
    if (call == null) {
      throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
  }
```


最终调用的是OkHttp的enqueue方法进行网络请求，对返回结果进行转化
Response<T> parseResponse(okhttp3.Response rawResponse)
```java
***
serviceMethod.toResponse()
***

  R toResponse(ResponseBody body) throws IOException {
    return responseConverter.convert(body);
  }
```
调用了Converter的convert方法对返回转化
如Gson-Converter就是把返回结果进行解析生成实体类对象


同时在返回时调用retrofit.callbackExecutor
使返回结果在主线程。
