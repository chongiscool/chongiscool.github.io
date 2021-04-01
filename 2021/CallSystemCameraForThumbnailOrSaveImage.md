

1. 声明需要相机支持

```xml
<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
```

2. 拍照

```java
// request code
static final int REQUEST_IMAGE_CAPTURE = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    // 确保有app响应这个拍照的请求
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // 调用相机，并等待结果
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}
```

3. 简单方式取回拍照后的缩略图(thumbnail)

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
}
```

**注意**：官方文档说，`data`参数获取到的数据，用于作为icon显示比较好。毕竟是缩略图嘛。

4. 保存刚刚拍的照片

申请权限先；

```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```

用一个第三方库[EasyPermission](https://github.com/googlesamples/easypermissions)，去解决申请权限，参考官方文档或参考[这里](https://www.jianshu.com/p/0bbd2445f473)

##### 1. 照片仅对当前app可见

a. 通过使用`getExternalFilesDir()`获取到存储照片内部存储路径；举个例子有个app是**takepic**，它内部存储如下：`Android/data/com.deao.takepic`；文档中提到，**Android4.4**及其以后版本，访问内部存储不需要申请权限，反之，须声明。

b. 创建保存的图片文件名

```java
// 后续，用于添加到相册，需要这个图片的绝对路径
String mCurrentPhotoPath;

private File createImageFile() throws IOException {
    // Create an image file name
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    String imageFileName = "JPEG_" + timeStamp + "_";
    File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
    /*File storageDir = getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES);

        if (!storageDir.exists()) {
            Log.d("TAG", "createImageFile: " + storageDir.mkdir());
        }*/
    // image 名称大概是：JPEG_20181216_200254_34**53.jpeg
    File image = File.createTempFile(
        imageFileName,  /* prefix */
        ".jpg",         /* suffix */
        storageDir      /* directory */
    );

    // Save a file: path for use with ACTION_VIEW intents
    mCurrentPhotoPath = image.getAbsolutePath();
    return image;
}
```

创建Intent并调用它如下:

```java
static final int REQUEST_TAKE_PHOTO = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    // Ensure that there's a camera activity to handle the intent
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // Create the File where the photo should go
        File photoFile = null;
        try {
            photoFile = createImageFile();
        } catch (IOException ex) {
            // Error occurred while creating the File
            ...
        }
        // Continue only if the File was successfully created
        if (photoFile != null) {
            // authorities="com.deao.takepic.fileprovider"
            Uri photoURI = FileProvider.getUriForFile(this, "com.deao.takepic.fileprovider", photoFile);
            // MUST be thrown FileUriExposedException
            // Uri photoURI = Uri.from(photo_file_path);
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
            startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
        }
    }
}
```

为适配7.0(API 24)或更高版本，通常要用`FileProvider`来处理**路径**了。如果还像之前那样传一个类似`file://`的URI，则会报[FileUriExposedException](https://developer.android.com/reference/android/os/FileUriExposedException.html)。现在传的是`content://`的URL。去`AndroidManifest`文件配置 [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)

```xml
<application>
   ...
   <provider
        android:name="android.support.v4.content.FileProvider"
        android:authorities="com.deao.takepic.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths"></meta-data>
    </provider>
    ...
</application>
```

然后，到`res/xml/file_paths`查看下我们的文件路径配置；

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path name="my_images" path="Android/data/com.deao.takepic/files/Pictures" />
    <external-path name="my_images" path="Pictures" />
</paths>
```

**解释**：类似于对`FileProvider`的声明，以上声明了两个路径的请求内容的URI(request content URI)。` <external-files-path name="my_images" path="Android/data/com.deao.takepic/files/Pictures" />`，声明图片保存到**takepic** app的私人文件夹内，其他应用都不能访问；`<external-path name="my_images" path="Pictures" />`，声明到图片保存到外置存储器的`Pictures`目录下，对设备上所有app可见，当然，添加到相册也是可以的。

更多细节参考这里[FileProvider#SpecifyFiles](https://developer.android.com/reference/android/support/v4/content/FileProvider#SpecifyFiles)
下面再`onActivityResult`回调中等待结果。

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
    // galleryAddPic();
}

```

##### 2. 照片所有app可见
...




### 添加到相册(Gallery)
```java
// mCurrentPhotoPath = ""
private void galleryAddPic() {
    Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
    File f = new File(mCurrentPhotoPath);
    Uri contentUri = Uri.fromFile(f);
    mediaScanIntent.setData(contentUri);
    this.sendBroadcast(mediaScanIntent);
}
```

**[注意事项](https://developer.android.com/training/camera/photobasics)**：如果相片是存储该app到`getExternalFileDir()`目录下的，**Media Scanner**是无法访问的，也就无法添加到相册；其他app也无法访问。


参考：

[1. Camera#takephotos](https://developer.android.com/training/camera/photobasics)

[2. FileProvider#SpecifyFiles](https://developer.android.com/reference/android/support/v4/content/FileProvider#SpecifyFiles)