
# map

对Observable发送的事件逐个应用Function进行变换：

```java
Observable
        .range(0, 5)
        .map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) throws Exception {
                return "item " + integer;
            }
        })
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.e(TAG, "accept: " + s);
            }
        });
```

# flatMap

对初始Observable的每个事件进行一次变换，每个变换都会生成一个新的子Observable，然后Observer会接收到所有子Observable发送的事件，顺序不固定。

一生多，多加多，合一

```java
Observable
        .range(0, 5)
        .flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(Integer integer) throws Exception {
                String[] strArr = new String[integer];
                for (int i = 0; i < integer; i++) {
                    strArr[i] = "number " + integer + ", count = " + (i + 1);
                }
		// 单个原始事件衍生出一个新的Observable，系统会自动发送这个Observable的事件
                return Observable.fromArray(strArr);
            }
        })
        .subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.e(TAG, "onSubscribe: ");
            }

            @Override
            public void onNext(String s) {
	        // 接收到所有衍生新Observable的事件
                Log.e(TAG, "onNext: " + s);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {
                Log.e(TAG, "onComplete: ");
            }
        });
```

# concatMap

flatMap的有序版，按照初始Observable所发送事件的先后顺序发送子事件。

# switchMap

flatMap的特殊情况版，目前不知道具体区别

# concat

将多个Observable按顺序组合发射，前一个Observable必须调用onComplete后，下一个Observable才能继续发送事件。只有最后一个Observable调用的onComplete才会触发Observer的onComplete回调，前面的Observable调用的都是用来让下一个Observable开始发送事件。

可以用在先从缓存读取数据，若无再从网络请求数据的场景下，适用于前后顺序很关键的场景下。


```java
final Observable<Long> observableTimer = Observable.timer(2, TimeUnit.SECONDS);
final Observable<String> observableTwo = Observable.just("hello", "rxjava2");
Observable.concat(observableTimer, observableTwo)
        .subscribe(new Observer<Object>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.e(TAG, "onSubscribe: ");
            }

            @Override
            public void onNext(Object s) {
                Log.e(TAG, "onNext: " + s);
            }

            @Override
            public void onError(Throwable e) {
                Log.e(TAG, "onError: " + e);
            }

            @Override
            public void onComplete() {
                Log.e(TAG, "onComplete: ");
            }
        });

```

# zip

将多个Observable中对应位置的事件通过指定方式进行组合，然后发送合并后的时间，缺少配对的事件则不进行发送，例如一个发送两个事件，一个发送三个事件，则最后只会合并发送出两个事件。

zip操作符适用于需要将多方面数据操作结果组合使用的情况。

```java
final Observable<String> observableOne = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("one 1");
        emitter.onNext("one 2");
        emitter.onNext("one 3");
        emitter.onComplete();
    }
});

final Observable<String> observableTwo = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("two 1");
        emitter.onNext("two 2");
        emitter.onComplete();
    }
});
Observable
        .zip(observableOne, observableTwo, new BiFunction<String, String, String>() {
            @Override
            public String apply(String s, String s2) throws Exception {
                return s + " ------ " + s2;
            }
        })
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.e(TAG, "accept: " + s);
            }
        });

```

# interval
间隔固定时间触发，适用于需要重复执行的场景，例如消息轮询

```java
private Disposable disposable;

private void initData() {

    final Observable<Long> interval = Observable.interval(1, TimeUnit.SECONDS);
    interval.subscribe(new Observer<Long>() {

        @Override
        public void onSubscribe(Disposable d) {
            disposable = d;
            Log.e(TAG, "onSubscribe: ");
        }

        @Override
        public void onNext(Long aLong) {
            Log.e(TAG, "onNext: " + aLong);
        }

        @Override
        public void onError(Throwable e) {
            Log.e(TAG, "onError: " + e);
        }

        @Override
        public void onComplete() {
            Log.e(TAG, "onComplete: ");
        }
    });
}

@Override
protected void onDestroy() {
    super.onDestroy();
    disposable.dispose();
}

```
