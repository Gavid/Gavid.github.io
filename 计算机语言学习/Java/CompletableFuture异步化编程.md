## CompletableFuture异步化编程

### CompletableFuture实现出现异常后，重试操作

```java 
public CompletableFuture<String> cutWordExpandAsync(AnalyzerType analyzerType, String s) {
  AtomicInteger retry = new AtomicInteger(this.retry);
  CompletableFuture<String> future = new CompletableFuture<>();
  AsyncReceiveHandler handler = new AsyncReceiveHandler() {
    @Override
    public void callBack(Object o, Object result) {
      future.complete((String) result);
    }
  };
  ProxyFactory.setThreadLocalContextAndAsyncRecvHandler("context", handler);
  map.get(ASYNC).get(analyzerType).cutWordExpand(s);
  return future.whenComplete((result, e) -> {
    if (e == null){
      future.complete(result);
    }else{
      if (retry.decrementAndGet() > 0){
        cutWordExpandAsync(analyzerType, s);
      }else{
        future.completeExceptionally(new AnalyzeException());
      }
    }
  });
}

```

### AsyncUtil 用于对CompletableFuture.allOf()无返回值的封装处理

```java
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.stream.Collectors;

public class AsyncUtil {
    public static  <T> CompletableFuture<List<T>> allOf(List<CompletableFuture<T>> futuresList) {
        CompletableFuture<Void> allFuturesResult =
                CompletableFuture.allOf(futuresList.toArray(new CompletableFuture[futuresList.size()]));
        return allFuturesResult.thenApplyAsync(v ->
                futuresList.stream().
                        map(future -> future.join()).
                        collect(Collectors.<T>toList())
        );
    }
}
```

### 参考文章

（推荐）Java CompletableFuture 详解：https://colobu.com/2016/02/29/Java-CompletableFuture/

[CompletableFuture详解](https://segmentfault.com/a/1190000039721242)

[Java8的CompletionService使用与原理](https://www.cnblogs.com/shijiaqi1066/p/10454237.html)

[使用CompletableFuture优化你的代码执行效率](https://www.cnblogs.com/fingerboy/p/9948736.html)

CompletableFuture组合异步编程详解：https://blog.csdn.net/lixinkuan328/article/details/89944378
（推荐）CompletableFuture 组合处理 allOf 和 anyOf太赞了！：https://cloud.tencent.com/developer/article/1834779

廖雪峰学习网站：https://www.liaoxuefeng.com/wiki/1252599548343744/1306581182447650

[Java实现异步调用](https://www.cnblogs.com/sword-successful/p/11181714.html)


SpringBoot 中异步执行任务的 2 种方式：http://ckjava.com/2019/08/07/Spring-Boot-EnableAsync-Async-ExecutorService-usage-practice/
