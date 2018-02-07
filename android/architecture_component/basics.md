1. ViewModel：绑定到具体生命周期，用来存取数据。一般用来保存View的数据，与其他部分进行沟通，例如数据仓库或者domain层。
2. LifecycleOwner/LifecyclerRegistryOwner：AppCompatActivity和Support库Fragment都实现了这两个接口。
3. LiveData：可以在多个组件中观测数据变化，同时遵循LifecycleOwner的生命周期规律。

# 开发实例：
## ViewModel结合LiveData：
```java
// 1. 初始状态，旋转屏幕重新计时
public class ChronoActivity1 extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Chronometer chronometer = findViewById(R.id.chronometer);

        chronometer.start();
    }
}

// 2. 使用ViewModel绑定到Activity上，保存数据
public class ChronometerViewModel extends ViewModel {
    @Nullable
    private Long mStartTime;

    @Nullable
    public Long getStartTime() { return mStartTime; }

    public void setStartTime(final long startTime) { mStartTime = startTime; }
}

public class ChronoActivity2 extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_main);
        // 将ViewModel绑定到Activity上
	ChronometerViewModel chronometerViewModel
	    = ViewModelProviders.of(this).get(ChronometerViewModel.class);
	
	Chronometer chronometer = findViewById(R.id.chronometer);

	if (chronometerViewModel.getStartTime() == null) {
            long startTime = SystemClock.elapsedRealtime();
	    chronometerViewModel.setStartTime(startTime);
	    chronometer.setBase(startTime);
	} else {
            chronometer.setBase(chronometerViewModel.getStartTime());
	}

	chronometer.start();
    }
}

// 3. 结合LiveData
public class LiveDataTimerViewModel extends ViewModel {

    private static final int ONE_SECOND = 1000;
    // ViewModel中包含LiveData
    private MutableLiveData<Long> mElapsedTime = new MutableLiveData<>();

    private long mInitialTime;

    public LiveDataTimerViewModel() {
        mInitialTime = SystemClock.elapsedRealtime();
        Timer timer = new Timer();

        // Update the elapsed time every second.
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                final long newValue = (SystemClock.elapsedRealtime() - mInitialTime) / 1000;
		// LiveData更新数据，供观测者接收
                mElapsedTime.postValue(newValue);
            }
        }, ONE_SECOND, ONE_SECOND);

    }

    @SuppressWarnings("unused")  // Will be used when step is completed
    public LiveData<Long> getElapsedTime() {
        return mElapsedTime;
    }
}


public class ChronoActivity3 extends AppCompatActivity {

    private LiveDataTimerViewModel mLiveDataTimerViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.chrono_activity_3);
        // 绑定ViewModel
        mLiveDataTimerViewModel = ViewModelProviders.of(this).get(LiveDataTimerViewModel.class);

        subscribe();
    }

    private void subscribe() {
        final Observer<Long> elapsedTimeObserver = new Observer<Long>() {
            @Override
            public void onChanged(@Nullable final Long aLong) {
                String newText = ChronoActivity3.this.getResources().getString(
                        R.string.seconds, aLong);
                ((TextView) findViewById(R.id.timer_textview)).setText(newText);
                Log.d("ChronoActivity3", "Updating timer");
            }
        };
        // 在Activity中监听LiveData的变化
        mLiveDataTimerViewModel.getElapsedTime().observe(this,elapsedTimeObserver);
    }
}


```

## 创建Lifecycle aware组件
```
public class BoundLocationManager {
    public static void bindLocationListenerIn(LifecycleOwner lifecycleOwner,
                                              LocationListener listener, Context context) { // 传入生命周期主体，处理器，上下文
        new BoundLocationListener(lifecycleOwner, listener, context);
    }

    @SuppressWarnings("MissingPermission")
    static class BoundLocationListener implements LifecycleObserver {
        private final Context mContext;
        private LocationManager mLocationManager;
        private final LocationListener mListener;

        public BoundLocationListener(LifecycleOwner lifecycleOwner,
                                     LocationListener listener, Context context) {
            mContext = context;
            mListener = listener;
	    // 当前组件观察生命周期
            lifecycleOwner.getLifecycle().addObserver(this);
        }

        // onResume事件触发方法
        @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
        void addLocationListener() {
            mLocationManager =
                    (LocationManager) mContext.getSystemService(Context.LOCATION_SERVICE);
            mLocationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, mListener);
            Log.d("BoundLocationMgr", "Listener added");

            Location lastLocation = mLocationManager.getLastKnownLocation(
                    LocationManager.GPS_PROVIDER);
            if (lastLocation != null) {
                mListener.onLocationChanged(lastLocation);
            }
        }

        // onPause事件触发方法
        @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
        void removeLocationListener() {
            if (mLocationManager == null) {
                return;
            }
            mLocationManager.removeUpdates(mListener);
            mLocationManager = null;
            Log.d("BoundLocationMgr", "Listener removed");
        }
    }
}

public class LocationActivity extends AppCompatActivity {

    private static final int REQUEST_LOCATION_PERMISSION_CODE = 1;

    private LocationListener mGpsListener = new MyLocationListener();

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
            @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (grantResults[0] == PackageManager.PERMISSION_GRANTED
                && grantResults[1] == PackageManager.PERMISSION_GRANTED) {
            bindLocationListener();
        } else {
            Toast.makeText(this, "This sample requires Location access", Toast.LENGTH_LONG).show();
        }
    }

    private void bindLocationListener() {
        // 绑定监听器
        BoundLocationManager.bindLocationListenerIn(this, mGpsListener, getApplicationContext());
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.location_activity);

        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)
                != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(this,
                Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.ACCESS_FINE_LOCATION,
                            Manifest.permission.ACCESS_COARSE_LOCATION},
                    REQUEST_LOCATION_PERMISSION_CODE);
        } else {
            bindLocationListener();
        }
    }

    private class MyLocationListener implements LocationListener {
        @Override
        public void onLocationChanged(Location location) {
            TextView textView = findViewById(R.id.location);
            textView.setText(location.getLatitude() + ", " + location.getLongitude());
        }
    }
}
```

# 注意事项
- LiveData的setValue()不能从子线程调用，如果要在子线程中更新数据，需要使用postVale()。

- ViewModel根据ViewModelProviders.of()中绑定的生命周期主体来生成，绑定同个主体就使用同个ViewModel，一个主体一个ViewModel。
