### 选择多张
- [Android: 关于系统相册多选图片的问题 不知道行不行](https://blog.csdn.net/Number____10/article/details/105483227)
	- 经试验可以选择多张,但问题在于: 没有办法限制可以选择的数量,大部分场景是现在最多选择N张照片
    - getData()和getClipData()返回的类型是不一样的，如果返回的多张图片的地址，必须用getClipData()来处理。此时getData()返回的是null。getClipData()返回的类型可以理解为getData()返回类型的list
      ```java
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
          try {
              Intent intent = new Intent();
              intent.setType("image/*");
              intent.putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true);
              intent.setAction(Intent.ACTION_GET_CONTENT);
              startActivityForResult(Intent.createChooser(intent, "Select Picture"), REQUEST_CODE_IMAGE);
          }catch(Exception e){
              Intent photoPickerIntent = new Intent(this, XYZ.class);
              startActivityForResult(photoPickerIntent, REQUEST_CODE_IMAGE);
          }
      } else {
          Intent photoPickerIntent = new Intent(this, XYZ.class);
          startActivityForResult(photoPickerIntent, REQUEST_CODE_IMAGE);
      }

      protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
          super.onActivityResult(requestCode, resultCode, data);
          Uri uri;
          List<String> fileList = new ArrayList<>();
          if (requestCode == REQUEST_CODE_IMAGE && data != null) {
              ClipData imageNames = data.getClipData();
              if (imageNames != null) {
                  for (int i = 0; i < imageNames.getItemCount(); i++) {
                      Uri imageUri = imageNames.getItemAt(i).getUri();
                      fileList.add(imageUri.toString());
                  }
              } else {
                  uri = data.getData();
                  fileList.add(uri.toString());
              }
          } else {
              uri = data.getData();
              fileList.add(uri.toString());
          }
          if(!fileList.isEmpty()){
              for(String item:fileList){
                  Log.d("testCreateFeedback","iamge item: " + item);
              }
          }
      }
      ```

- [Android 本地图片多选 知乎自己实现的开源库](https://blog.csdn.net/u014608640/article/details/82051939)
- [Android 一起来看看知乎开源的图片选择库](https://juejin.im/post/6844903516893347854)

### 打开相机及相册
- [Android 调用相机拍照，适配到Android 10](https://juejin.im/post/6844903944095793166)
- [android从相册选择图片和拍照选择图片](https://www.jianshu.com/p/f13dfd5a823e)
- [Android开发从相册中选取照片](https://www.jianshu.com/p/763c181694a4)
- [Android 调用相机、相册功能](https://www.cnblogs.com/LEON-D/p/11389346.html)

### 打开相册
```java
private void choosePicture(){
  //select single image via original api
  //经试验
  //使用 Intent.ACTION_PICK:
  //  查询数据库,可以获取到文件路径;
  //  通用的根据URI获取文件路径方法,可以获取到文件路径;
  //使用 Intent.ACTION_GET_CONTENT:
  //  查询数据库,无法获取到文件路径
  //  通用的根据URI获取文件路径方法,可以获取到文件路径;
  //Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
  Intent intent = new Intent(Intent.ACTION_PICK);
  intent.setType("image/*");
  startActivityForResult(intent, REQUEST_CODE_IMAGE);
}
```

### 根据Uri获取图片路径
1. 直接查询数据库获得
    ```java
    public static String gainImagePathFromUri(Context context, Uri imageUri) {
        String[] filePathColumn = {MediaStore.Images.Media.DATA};
        Cursor cursor = context.getContentResolver().query(imageUri,
                filePathColumn, null, null, null);//从系统表中查询指定Uri对应的照片
        cursor.moveToFirst();
        int columnIndex = cursor.getColumnIndex(filePathColumn[0]);
        String path = cursor.getString(columnIndex);  //获取照片路径
        cursor.close();
        return path;
    }
    ```
2. 通用的根据URI获取文件路径方法,不限于图片
    ```java
    @TargetApi(Build.VERSION_CODES.KITKAT)
    public static String gainPathFromUri(final Context context, final Uri uri) {
        // DocumentProvider
        if (Platform.hasKitKat() && DocumentsContract.isDocumentUri(context, uri)) {
            // ExternalStorageProvider
            if (isExternalStorageDocument(uri)) {
                final String docId = DocumentsContract.getDocumentId(uri);
                final String[] split = docId.split(":");
                final String type = split[0];

                if ("primary".equalsIgnoreCase(type)) {
                    return Environment.getExternalStorageDirectory() + "/" + split[1];
                }

                // TODO handle non-primary volumes
            } else if (isDownloadsDocument(uri)) { // DownloadsProvider

                final String id = DocumentsContract.getDocumentId(uri);
                final Uri contentUri = ContentUris.withAppendedId(
                        Uri.parse("content://downloads/public_downloads"), Long.valueOf(id));

                return getDataColumn(context, contentUri, null, null);
            } else if (isMediaDocument(uri)) { // MediaProvider
                final String docId = DocumentsContract.getDocumentId(uri);
                final String[] split = docId.split(":");
                final String type = split[0];

                Uri contentUri = null;
                if ("image".equals(type)) {
                    contentUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
                } else if ("video".equals(type)) {
                    contentUri = MediaStore.Video.Media.EXTERNAL_CONTENT_URI;
                } else if ("audio".equals(type)) {
                    contentUri = MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
                }

                final String selection = "_id=?";
                final String[] selectionArgs = new String[]{
                        split[1]
                };

                return getDataColumn(context, contentUri, selection, selectionArgs);
            }
        } else if ("content".equalsIgnoreCase(uri.getScheme())) { // MediaStore (and general)
            return getDataColumn(context, uri, null, null);
        } else if ("file".equalsIgnoreCase(uri.getScheme())) { // File
            return uri.getPath();
        }

        return null;
    }

    /**
     * Get the value of the data column for this Uri. This is useful for
     * MediaStore Uris, and other file-based ContentProviders.
     *
     * @param context       The context.
     * @param uri           The Uri to query.
     * @param selection     (Optional) Filter used in the query.
     * @param selectionArgs (Optional) Selection arguments used in the query.
     * @return The value of the _data column, which is typically a file path.
     */
    public static String getDataColumn(Context context, Uri uri, String selection,
                                       String[] selectionArgs) {

        Cursor cursor = null;
        final String column = "_data";
        final String[] projection = {
                column
        };

        try {
            cursor = context.getContentResolver().query(uri, projection, selection, selectionArgs, null);
            if (cursor != null && cursor.moveToFirst()) {
                final int columnIndex = cursor.getColumnIndexOrThrow(column);
                return cursor.getString(columnIndex);
            }
        } finally {
            if (cursor != null)
                cursor.close();
        }
        return null;
    }


    /**
     * @param uri The Uri to check.
     * @return Whether the Uri authority is ExternalStorageProvider.
     */
    public static boolean isExternalStorageDocument(Uri uri) {
        return "com.android.externalstorage.documents".equals(uri.getAuthority());
    }

    /**
     * @param uri The Uri to check.
     * @return Whether the Uri authority is DownloadsProvider.
     */
    public static boolean isDownloadsDocument(Uri uri) {
        return "com.android.providers.downloads.documents".equals(uri.getAuthority());
    }

    /**
     * @param uri The Uri to check.
     * @return Whether the Uri authority is MediaProvider.
     */
    public static boolean isMediaDocument(Uri uri) {
        return "com.android.providers.media.documents".equals(uri.getAuthority());
    }
    ```
