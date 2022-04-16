---
Author: pony@diynova.com
Date: 2022-03-24 17:18:56
LastEditors: pony@diynova.com
LastEditTime: 2022-04-16 18:37:50
FilePath: /notes/docs/android/camera/takephoto.md
Description: takephoto
---

# 相机使用

## 1. manifest 配置
> 如果您的应用使用相机，但不需要相机也可以正常运作，应将 android:required 设为 false。这样，Google Play 便会允许未装有相机的设备下载您的应用。因此，您必须负责通过调用 hasSystemFeature(PackageManager.FEATURE_CAMERA_ANY) 检查相机在运行时的可用性。如果相机不可用，您应停用相机功能。

```
 <manifest>
      <uses-feature android:name="android.hardware.camera"
                    android:required="true" />
  </manifest>
```

## 2. 拍照
> Android 向其他应用委托操作的方法是调用一个 Intent，用于描述您要执行的操作。此过程涉及三个部分：**Intent 本身**，**用于启动外部 Activity 的调用**，以及用于**在焦点返回到 Activity 时处理图片**数据的一些代码。
```
   val REQUEST_IMAGE_CAPTURE = 1

    private fun dispatchTakePictureIntent() {
        Intent(MediaStore.ACTION_IMAGE_CAPTURE).also { takePictureIntent ->
            takePictureIntent.resolveActivity(packageManager)?.also {
                startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE)
            }
        }
    }
    
```

## 3. 获取缩略图
> Android 相机应用会对返回 Intent（作为 extra 中的小型 Bitmap 传递给 onActivityResult()，使用键 "data"）中的照片进行编码。下面的代码会检索此图片，并将其显示在一个 ImageView 中。

```
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        val imageBitmap = data.extras.get("data") as Bitmap
        imageView.setImageBitmap(imageBitmap)
    }
}
```

## 4. 保存完整尺寸的照片
> Android 相机应用会保存一张完整尺寸的照片，前提是您为该照片指定了一个文件来保存它。您必须为相机应用应保存照片的位置提供一个完全限定的文件名称。 但是如果获得完整尺寸的照片的话，intent 中就不会有值，为 null，需要自己手动保存图片

> **存放路径公共**: 通常情况下，用户使用设备相机拍摄的所有照片都应保存在设备的公共外部存储设备中，以供所有应用访问。合适的共享照片目录由具有 DIRECTORY_PICTURES 参数的 getExternalStoragePublicDirectory() 提供。由于这种方法提供的目录会在所有应用之间共享，因此对该目录执行读写操作分别需要 READ_EXTERNAL_STORAGE 和 WRITE_EXTERNAL_STORAGE 权限。拥有写入权限即暗示可以读取，因此，如果您需要写入到外部存储设备，就只需请求一个权限：

```xml
<manifest>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
</manifest>
```

> **存放路径私有**: 不过，如果您希望这些照片仍仅对您的应用保持私密状态，可以改用 getExternalFilesDir() 提供的目录。在 Android 4.3 及更低版本中，向此目录写入还需要 WRITE_EXTERNAL_STORAGE 权限。从 Android 4.4 开始不再需要此权限，因为其他应用无法访问该目录，因此您可以通过添加 maxSdkVersion 属性来声明应仅在较低版本的 Android 上请求此权限：

```xml
<manifest>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                      android:maxSdkVersion="18" />
    
</manifest>
```
> 注意：用户卸载您的应用后，您在 getExternalFilesDir() 或 getFilesDir() 提供的目录中保存的文件会被删除。


确定文件目录后，您需要指定一个不会跟其他文件产生冲突的文件名。您还可以将该路径保存在成员变量中以供日后使用。下面是某个方法中的示例解决方案，针对使用日期-时间戳的新照片返回一个独一无二的文件名。

```
    lateinit var currentPhotoPath: String

    @Throws(IOException::class)
    private fun createImageFile(): File {
        // Create an image file name
        val timeStamp: String = SimpleDateFormat("yyyyMMdd_HHmmss").format(Date())
        val storageDir: File = getExternalFilesDir(Environment.DIRECTORY_PICTURES)
        return File.createTempFile(
                "JPEG_${timeStamp}_", /* prefix */
                ".jpg", /* suffix */
                storageDir /* directory */
        ).apply {
            // Save a file: path for use with ACTION_VIEW intents
            currentPhotoPath = absolutePath
        }
    }
    
```

使用这种方法为照片创建文件后，您现在可以创建和调用 Intent，如下所示：
```
    val REQUEST_TAKE_PHOTO = 1

    private fun dispatchTakePictureIntent() {
        Intent(MediaStore.ACTION_IMAGE_CAPTURE).also { takePictureIntent ->
            // Ensure that there's a camera activity to handle the intent
            takePictureIntent.resolveActivity(packageManager)?.also {
                // Create the File where the photo should go
                val photoFile: File? = try {
                    createImageFile()
                } catch (ex: IOException) {
                    // Error occurred while creating the File
                    
                    null
                }
                // Continue only if the File was successfully created
                photoFile?.also {
                    val photoURI: Uri = FileProvider.getUriForFile(
                            this,
                            "com.example.android.fileprovider",
                            it
                    )
                    takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI)
                    startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO)
                }
            }
        }
    }
    
```
> * 注意：我们使用的是 getUriForFile(Context, String, File)，它会返回 content:// URI。对于以 Android 7.0（API 级别 24）及更高版本为目标平台的最新应用，跨越软件包边界传递 file:// URI 会导致出现 FileUriExposedException。因此，我们现在介绍一种更通用的方法来使用 FileProvider 存储图片。

现在，您需要配置 FileProvider。在应用清单中，向您的应用添加提供器：
```
<application>
  
  <provider
        android:name="android.support.v4.content.FileProvider"
        android:authorities="com.example.android.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths"></meta-data>
    </provider>
  
</application>
```

> 确保授权字符串与 `getUriForFile(Context, String, File)` 的第二个参数匹配。在提供器定义的元数据部分中，您可以看到提供器需要在专用资源文件 `res/xml/file_paths.xml` 中配置符合条件的路径。下面显示了此特定示例所需的内容：
```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path name="my_images" path="Android/data/com.example.package.name/files/Pictures" />
</paths>
```

路径组件对应于 getExternalFilesDir() 返回的路径（在调用 Environment.DIRECTORY_PICTURES 时）。请务必将 com.example.package.name 替换为应用的实际软件包名称。此外，请参阅 [FileProvider](https://developer.android.com/reference/androidx/core/content/FileProvider) 的文档，查看除 external-path 之外您可以使用的路径说明符的详细说明。



## 5. 将照片添加到图库

通过 Intent 制作照片时，您应该知道图片所在的位置，因为您一开始就指定了保存该图片的位置。对于所有其他人而言，为了让您的照片可供访问，最简单的方式可能是让其可从系统的媒体提供商访问。

> 注意：如果您将照片保存到 getExternalFilesDir() 提供的目录中，媒体扫描器将无法访问相应的文件，因为这些文件对您的应用保持私密状态。

下面的示例方法演示了如何调用系统的媒体扫描器以将您的照片添加到媒体提供商的数据库中，使 Android 图库应用中显示这些照片并使它们可供其他应用使用。

```
    private fun galleryAddPic() {
        Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE).also { mediaScanIntent ->
            val f = File(currentPhotoPath)
            mediaScanIntent.data = Uri.fromFile(f)
            sendBroadcast(mediaScanIntent)
        }
    }
```

## 6. 对调整后的图片进行解码
如果内存有限，管理多张完整尺寸的图片可能会比较棘手。如果您发现应用仅显示几张图片就内存不足，则可以将 JPEG 扩展到内存数组（已调整为与目标视图的大小匹配），从而大幅减少动态堆的使用量。下面的示例方法演示了这种方法。
```
    private fun setPic() {
        // Get the dimensions of the View
        val targetW: Int = imageView.width
        val targetH: Int = imageView.height

        val bmOptions = BitmapFactory.Options().apply {
            // Get the dimensions of the bitmap
            inJustDecodeBounds = true

            val photoW: Int = outWidth
            val photoH: Int = outHeight

            // Determine how much to scale down the image
            val scaleFactor: Int = Math.min(photoW / targetW, photoH / targetH)

            // Decode the image file into a Bitmap sized to fill the View
            inJustDecodeBounds = false
            inSampleSize = scaleFactor
            inPurgeable = true
        }
        BitmapFactory.decodeFile(currentPhotoPath, bmOptions)?.also { bitmap ->
            imageView.setImageBitmap(bitmap)
        }
    }
    
```