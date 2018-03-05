原生的权限管理机制：

- 首次请求，跳出权限请求对话框，可允许可拒绝。
- 若首次拒绝进行二次请求，此时`ActivityCompat.shouldRequestPermissionRationale()`为true，同时请求对话框出现Don't ask again选项。
- 若勾选Don't ask again，第三次请求不会出现对话框，直接回调拒绝。

```java
public class MainActivity extends AppCompatActivity {
    private static final int REQUEST_PERMISSION_READ_CONTACTS = 980;
    private static final String TAG = "MainActivity";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (ActivityCompat.checkSelfPermission(this,
                Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED) {
            // doesn't have the permission
            if (ActivityCompat.shouldShowRequestPermissionRationale(this,
                    Manifest.permission.READ_CONTACTS)) {
                // The permission request has been denied once. Should show rationale and request again
                Toast.makeText(this, "This app needs to read contacts to work.", Toast.LENGTH_SHORT).show();
                // request Again
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.READ_CONTACTS}, REQUEST_PERMISSION_READ_CONTACTS);
            } else {
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.READ_CONTACTS}, REQUEST_PERMISSION_READ_CONTACTS);
            }
        } else {
            // already have the permission
            Log.e(TAG, "onCreate: already have the permission");
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            // permission was granted
            Log.e(TAG, "onRequestPermissionsResult: granted");
        } else {
            // permission was denied.
            Log.e(TAG, "onRequestPermissionsResult: denied");
        }
    }
}
```
