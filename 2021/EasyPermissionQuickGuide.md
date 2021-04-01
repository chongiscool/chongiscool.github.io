> Android 6.0之后需要运行时权限了， 更多具体概念请查阅[Android开发者网站](https://developer.android.google.cn/training/permissions/requesting)

使用[EasyPermissions](https://github.com/googlesamples/easypermissions)这个第三方库，以申请**存储权限**为例，有如下四个步骤；

1. 声明权限

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
申请同组的一个，被授予后；当同组的另外一个权限需要使用时，仅当`AndroidManifest`文件声明了该权限，系统会自动赋予，开发者无须再次申请。举个例子: 读和写sd两个权限是同一组的，当用户在app中授予了**读取**sd的权限，那当app需要用到**写入**sd的权限时，app会被Android系统自动授予，类似于family plan。

2. 导入第三方库

```
// build.gradle (app)
dependencies {
    implementation "pub.devrel:easypermissions:$easy_permission_ver"
}
```

3. 实现两个回调，跑log

```java
public class MainActivity extends AppCompatActivity implements EasyPermissions.PermissionCallbacks, EasyPermissions.RationaleCallbacks {
...
@Override
    public void onPermissionsGranted(int requestCode, @NonNull List<String> perms) {
        if (requestCode == 200) {
            Log.d("TAG", "onPermissionsGranted: ");
            dispatchTakePicIntent();
//            startCamera();
        }
    }

    @Override
    public void onPermissionsDenied(int requestCode, @NonNull List<String> perms) {
        if (requestCode == 200) {
            Log.d("TAG", "onPermissionsDenied: ");
        }
    }

    @Override
    public void onRationaleAccepted(int requestCode) {
        if (requestCode == 200) {
            Log.d("TAG", "onRationaleAccepted: ");
        }
    }

    @Override
    public void onRationaleDenied(int requestCode) {
        Log.d("TAG", "onRationaleDenied: ");
    }
```

Activity/Fragment中需要实现`onRequestPermissionsResult()`回调，不然有**诡异**的事发生。

举个例子，当用户第一次拒绝了app使用**sd存储器**的请求，**再次申请**时，出现向用户解释，为什么需要该权限理由，如：`need storage for store user info`。此时，不管用户选择授予，还是拒绝，都没有对应的log。

```java
@Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, this);
    }
```

4. 组装

设想，点击**拍照**按钮，申请权限，用户决定授予权限与否的情景。

```java
@OnClick(R.id.btn_take_pic)
    public void onCaptured() {
        // 检查外置存储器写权限
        if (!EasyPermissions.hasPermissions(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)) {
            // Ask permission with Permission Reasons, RequestCode and PermissionName
            EasyPermissions.requestPermissions(this, "need use storage", 200, Manifest.permission.WRITE_EXTERNAL_STORAGE);
            Log.d("TAG", "onCaptured: requestPermissions ");
        } else {
//            startCamera();
            dispatchTakePicIntent();
            Log.d("TAG", "onCaptured: hasPermissions");
        }
        Log.d("TAG", "onCaptured: ");
    }
```

**注意事项**：

1. 在`onPermissionsDenied`和`onRationaleDenied`回调方法中直接继续**申请权限**，**不起作用**的。
2. 在`onRationaleAccepted`回调方法中直接继续**申请权限**，不需要的，这方法走完后，会自动再次申请权限的。

Refs：
[EasyPermissions](https://github.com/googlesamples/easypermissions)