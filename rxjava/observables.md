创建的Observable可以给多个Observe观测

# 快捷发送
快捷发送一定内容的数据，接收方在onNext()回调中接收到数据，发送完毕自动调用onComplete()。
## just
按顺序不处理直接发送传递的事件，一次性最多发送10个：

```java
Observable
    .just("Hello", "world")
    .subscribe(new Consumer<String>() {
        @Override
        public void accept(String s) throws Exception {
            Log.e(TAG, "accept: " + s);
        }
    });
```

## fromArray
和just类似，只不过不限制发送个数，同时接受数组

```java
String[] arr = {"Hello","world","rxjava2"};
Observable
//      .fromArray("Hello","world","rxjava2")
        .fromArray(arr)
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.e(TAG, "accept: " + s);
            }
        });
```

## fromIterable
从实现Iterable接口的数据中获取，例如集合：

```java
List<String> list = new ArrayList<>();
Observable
        .fromIterable(list)
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.e(TAG, "accept: " + s);
            }
        });
```

## fromCallable
可以在call()中执行任意操作，返回一个Object，并且可以抛出异常

```java
Observable
        .fromCallable(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return someOperation();
            }
        })
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.e(TAG, "accept: " + s);
            }
        });

```

# 完整调用Observable.create()
ObservableOnSubscribe是一个接口，当有Observer订阅该Observable时，就会调用subscribe()。subscribe()中使用ObservableEmitter去发送事件，记得事件发送完毕需要调用onComplete()或者onError()。
```java
Observable
        .create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("Hello");
                emitter.onNext("RxJava2");
                emitter.onComplete();
            }
        })
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.e(TAG, "accept: " + s);
            }
        });
```

# 延迟发送
正常Observable是还未subscribe就创建完成了，例如：

```java
final Consumer<String> firstSubscriber = new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.e(TAG, "first onNext: " + s);
    }
};
final Consumer<String> secondSubscriber = new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.e(TAG, "second onNext: " + s);
    }
};
final Observable<String> observable = Observable.just("hello!" + System.currentTimeMillis());
observable.subscribe(firstSubscriber);
try {
    Thread.sleep(500);
    observable.subscribe(secondSubscriber);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```
此时打印出来的时间戳是一致的，因为是同一个Observable。

defer()在有观测者去subscribe的时候才创建Observable，打印出来的时间戳不一样，因为是不同的Observable。可以用于需要实时生成Observable的场景：

```java
final Observable<String> observable = Observable.defer(new Callable<ObservableSource<? extends String>>() {
    @Override
    public ObservableSource<? extends String> call() throws Exception {
        return Observable.just("hello!" + System.currentTimeMillis());
    }
});
observable.subscribe(firstSubscriber);
try {
    Thread.sleep(500);
    observable.subscribe(secondSubscriber);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

## 发送固定范围整数

```java
// 发送从0开始10个整数，步长为1，也就是0-9
Observable<Integer> observable = Observable.range(0, 10);

// 发送从0开始10个整数，步长为1，也就是0-9。初始延迟时间为2，间隔时间为2，时间单位为秒
Observable<Long> observable = Observable.intervalRange(0, 10, 2, 1, TimeUnit.SECONDS);
```

## 定时发送

```java
// 固定时间后发送`onNext(0L)`以及`onComplete()`事件
Observable<Long> timer = Observable.timer(1, TimeUnit.SECONDS);
```

## 特殊种类

```java
// empty()，直接调用onComplete()
Observable<Object> empty = Observable.empty();

// never()，不发送任何事件
Observable<Object> never = Observable.never();
```
# Observable的变体

Observable还有其他几种变体：Single、Completable、Maybe。

- Single：只能接收一个onNext事件，回调到onSuccess当中，没有onComplete

```java
Single
        .just("hello")
        .subscribe(new SingleObserver<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onSuccess(String s) {
                Log.e(TAG, "onSuccess: " + s);
            }

            @Override
            public void onError(Throwable e) {

            }
        });

```

- Completable：没有onNext事件，事件处理完成调用onComplete()，适用于没有返回值的操作：

```java
Completable
        .fromAction(new Action() {
            @Override
            public void run() throws Exception {

            }
        })
        .subscribe(new CompletableObserver() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onComplete() {

            }

            @Override
            public void onError(Throwable e) {

            }
        });

```

- Maybe：Completable和Single的综合体

```java
Maybe
        .fromSingle(new SingleSource<String>() {
            @Override
            public void subscribe(SingleObserver<? super String> observer) {

            }
        })
        .subscribe(new MaybeObserver<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onSuccess(String s) {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
```


