在多核设备上，多线程操作可以并行处理，提高速度。

## Thread和Runnable
Thread和Runnable是其他多线程工具类的基础，例如HandlerThread、AsyncTask、IntentService还有ThreadPoolExecutor。

```java
public class PhotoDecodeRunnable implements Runnable {
    @Override
    public void run() {
        // 把当前线程设为后台优先级，避免降低UI线程性能。
        android.os.Process.setThreadPriority(android.os.Process.THREAD_PRIORITY_BACKGROUND);
        // 保存当前线程，以便进行终止操作
	mPhotoTask.setImageDecodeThread(Thread.currentThread());
	...
    }
}
```

## Thread Pool
只需要执行一个任务的情况下可以直接使用Thread和Runnable，但是如果是其他情况，则可以根据需要选择合适的线程工具类。例如IntentService可以执行一系列任务，一次执行一个。而线程池可以同时执行多个线程。下面分析线程池的操作。

```java
public class PhotoManager {
    static {
        // 单实例模式，节省资源，方便管理
        sInstance = new PhotoManager();
    }
    private PhotoManager() {
        ...
    }

    static public PhotoTask startDownload(PhotoView imageView, boolean cacheFlag) {
        // 添加一个下载任务到线程池中待执行
        sInstance.mDownloadThreadPool.execute(downloadTask.getHTTPDownloadRunnable());
    }

    // 创建一个绑定到主线程的Handler
    mHandler = new Handler(Looper.getMainLooper()) {
        @Override
	public void handleMessage(Message inputMessage) {
	    // 运行在主线程中，可以进行UI操作
	}
    }
    
    // 系统可用核数，可以作为线程池初始大小和最大大小
    private static int NUMBER_OF_CORES = Runtime.getRuntime().availableProcessors();
    // 线程闲置后保持存活的时间
    private static final int KEEP_ALIVE_TIME = 1;
    private static final TimeUnit KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS;

    // 线程池接收Runnable的队列，可以根据需要选择具体实现
    private final BlockingQueue<Runnable> mDecodeWorkQueue;
    mDecodeWorkQueue = new LinkedBlockingQueue<Runnable>();

    // 创建线程池
    mDecodeThreadPool = new ThreadPoolExecutor(
            NUMBER_OF_CORES, // 初始线程池大小
	    NUMBER_OF_CORES, // 最大线程池大小
	    KEEP_ALIVE_TIME,
	    KEEP_ALIVE_TIME_UNIT,
	    mDecodeWorkQueue);

    // 将Runnable添加到线程池中执行
    public void handleState(PhotoTask photoTask, int state) {
        switch (state) {
	    //  下载完成后执行解码工作
            case DOWNLOAD_COMPLETE:
	        mDecodeThreadPool.execute(photoTask.getPhotoDecodeRunnable());
	}
    }

    // 取消任务
    public static void cancelAll() {
        Runnable[] runnableArray = new Runnable[mDecodeWorkQueue.size()];
	mDecodeWorkQueue.toArray(runnableArray);
	int len = runnableArray.length;
	// 因为Thread对象是由系统控制的，可能在app进程外被修改，所以需要加锁处理
	synchronized (sInstance) {
            for (int runnableIndex = 0; runnableIndex < len; runnableIndex++) {
                Thread thread = runnableArray[runnableIndex].mThread;
		if (null != thread) { // 如果线程非空，则中断掉。
                    thread.interrupt();
		}
	    }
	}
    }
}
```

### 发送执行结果到UI线程
```java
class PhotoDecodeRunnable implements RUnnable {
    PhotoDecodeRunnable(PhotoTask downloadTask) {
        mPhotoTask = downloadTask;
    }
    // 获取下载完毕的图像字节内容
    byte[] imageBuffer = mPhotoTask.getByteBuffer();
    
    public void run() {
        // 进行耗时操作
        returnBitmap = BitmapFactory.decodeByteArray(imageBuffer, 0, imageBuffer.length, bitmapOptions);

	...
        // 保存图像内容，因为在子线程中不能直接操作UI
	mPhotoTask.setImage(returnBitmap);
	// PhotoTask中进行处理
	mPhotoTask.handleDecodeState(DECODE_STATE_COMPLETED);
    }
}


public class PhotoTask {
    sPhotoManger = PhotoManager.getInstance();

    public void handleDecodeState(int state) {
        int outSTate;
	switch(state) {
            case PhotoDecodeRunnable.DECODE_STATE_COMPLETED:
	        outState = PhotoManager.TASK_COMPLETE;
		break;
	}

	handleState(outState);
    }

    void handleState(int state) {
        sPhotoManager.handleState(this, state);
    }
}

public class PhotoManager {
    public void handleState(PhotoTask photoTask, int state) {
        switch (state) {
            case TASK_COMPLETE:
	        Message completeMessage = mHandler.obtainMessage(state, photoTask);
		completeMessage.sendToTarget();
		break;
	}
    }

    mHandler = new Handler(Looper.getMainLooper()) {
        @Override
	public void handleMessage(Message inputMessage) {
            PhotoTask photoTask = (PhotoTask) inputMessage.obj;
	    PhotoView localView = photoTask.getPhotoView();

	    switch (inputMessage.what) {
                case TASK_COMPLETE:
		    localView.setImageBitmap(photoTask.getImage());
		    break;

		default:
		    super.handleMessage(inputMessage);
	    }
	}
    }
}
```

