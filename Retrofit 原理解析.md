**Retrofit 原理解析：**

* Retrofit 执行流程：
Retrofit 使用示例代码：
```
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://api.github.com")
                .addConverterFactory(GsonConverterFactory.create(gson))
                .build();
                
ApiService apiService = retrofit.create(ApiService.class);

apiService.getRepos("Muran-Hu")
                .enqueue(new Callback<List<Repo>>() {
                    @Override
                    public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
                        System.out.println("Success: " + gson.toJson(response.body()));
                    }

                    @Override
                    public void onFailure(Call<List<Repo>> call, Throwable t) {
                        System.out.println("Failed");
                    }
                });
```

源代码跟踪路径：
1. retrofit.create(ApiService.class) - create 方法为 Retrofit 核心方法
2. create() 方法中核心代码：
```
return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.adapt(okHttpCall);
          }
        });
```
3. 动态代理模式，将我们定义的 API 网络请求接口转换成 okhttp call：
```
public interface ApiService {
    @GET("/users/{user}/repos")
    Call<List<Repo>> getRepos(@Path("user") String user);
}
```
4. 动态代理中核心代码为：
```
ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
return serviceMethod.adapt(okHttpCall);
```
5. loadServiceMethod(method) - 该方法解析我们定义的API请求接口，封装成 ServiceMethod 对象，放入缓存中 `Map<Method, ServiceMethod<?, ?>> serviceMethodCache`
6. OkHttpCall - OkHttp 网络请求 Call
7. serviceMethod.adapt(okHttpCall) - 适配方法，将method转换成okHttpCall，并做线程切换
8. ServiceMethod 的 adapt 方法：
```
T adapt(Call<R> call) {
    return callAdapter.adapt(call);
}
```
9. callAdapter 为 ServiceMethod 中的属性：`CallAdapter<R, T> callAdapter;`
10. 下面要看 callAdapter 是如何初始化的：在第5步中 `loadServiceMethod(method)` 方法中
`ServiceMethod.Builder<>(this, method).build();` 的 build() 方法中初始化：
```
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
11. ServiceMethod.Builder<>().build():
```
callAdapter = createCallAdapter();
```
12. createCallAdapter() 方法中关键代码：
```
return (CallAdapter<T, R>) retrofit.callAdapter(returnType, annotations);
```
13. retrofit.callAdapter 方法中关键代码：
```
return nextCallAdapter(null, returnType, annotations);
```
14. nextCallAdapter 方法关键代码：
```
CallAdapter<?, ?> adapter = callAdapterFactories.get(i).get(returnType, annotations, this);
```
15. 现在要看 callAdapterFactories 何时初始化的，根据跟踪代码可知：
```
Retrofit retrofit = new Retrofit.Builder()
                ...
                .build();
```
16. Retrofit 内部类 Builder 的 build() 方法：
```
...
return new Retrofit(callFactory, baseUrl, unmodifiableList(converterFactories),
          unmodifiableList(callAdapterFactories), callbackExecutor, validateEagerly);
```
17. 调用 Retrofit 构造方法：
```
...
this.callAdapterFactories = callAdapterFactories;
...
```
可见，关键代码应该在Retrofit 内部类 Builder 的 build() 方法中是如何初始化 callAdapterFactories 的
18. Retrofit 内部类 Builder 的 build() 方法：
```
public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }

      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>(this.callAdapterFactories);
      callAdapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

      // Make a defensive copy of the converters.
      List<Converter.Factory> converterFactories =
          new ArrayList<>(1 + this.converterFactories.size());

      // Add the built-in converter factory first. This prevents overriding its behavior but also
      // ensures correct behavior when using converters that consume all types.
      converterFactories.add(new BuiltInConverters());
      converterFactories.addAll(this.converterFactories);

      return new Retrofit(callFactory, baseUrl, unmodifiableList(converterFactories),
          unmodifiableList(callAdapterFactories), callbackExecutor, validateEagerly);
    }
```
19. 18中核心代码：
```
Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>(this.callAdapterFactories);
      callAdapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));
```
20. 因为当前的 platform 为 Android，所以要找到 Android 类，看其中的 defaultCallbackExecutor 和 defaultCallAdapterFactory 返回的是啥对象
21. Android 为 Platform 的静态内部类：
```
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
22. 由此可见，是如何进行线程切换的。当我们调用代码进行网络请求时：
```
apiService.getRepos("Muran-Hu")
                .enqueue(new Callback<List<Repo>>() {
                    @Override
                    public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
                        System.out.println("Success: " + gson.toJson(response.body()));
                    }

                    @Override
                    public void onFailure(Call<List<Repo>> call, Throwable t) {
                        System.out.println("Failed");
                    }
                });
```
23. 此时 enqueue 方法会跳转到 ExecutorCallAdapterFactory -> ExecutorCallbackCall 的 enqueue 方法：
```
static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;
    final Call<T> delegate;

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
24. 而 ExecutorCallbackCall 中的 callbackExecutor 为19中的 MainThreadExecutor，而其中：
```
private final Handler handler = new Handler(Looper.getMainLooper());
```
25. 该Handler获取到 主线程即UI线程的 looper，由此可见 executor 执行完之后，会通过 handler 跳转回主线程，针对返回的数据进行 UI 处理。
