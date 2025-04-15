# 1 Bitmap内存占用计算  

Bitmap内存占用 = 宽度像素个数 x 高度像素个数 x 每个像素占用内存   

其中，每个像素占用内存和像素格式有关，参见：

```java
public static final Bitmap.Config ALPHA_8 //代表8位Alpha位图 每个像素占用1byte内存
public static final Bitmap.Config ARGB_4444 //代表16位ARGB位图 每个像素占用2byte内存
public static final Bitmap.Config ARGB_8888 //代表32位ARGB位图 每个像素占用4byte内存
public static final Bitmap.Config RGB_565 //代表8位RGB位图 每个像素占用2byte内存
```  

如果直接从SD卡读取一张图片，直接计算即可，如果是从密度目录中读取，那么还要考虑缩放比例  

举例：

那么对于一张2000x2000的ARGB_8888图片，在`res/drawable-xxhdpi`目录中，设备屏幕密度为`xxxhdpi`，内存占用计算：  

系统自动缩放比例：缩放比例 = 目标密度/资源密度 = 640dpi/480dpi = 1.333  

实际宽高：宽 = 2000/(640/480) = 1500 , 高 = 2000/(640/480) = 1500  

内存占用 = 1500 x 1500 x 4 = 9,000,000 byte ≈ 8.58 MB  

代码验证方式：  
```java
val bitmap = BitmapFactory.decodeResource(resources, R.drawable.your_image)
val byteCount = bitmap.allocationByteCount // 实际分配的内存大小
Log.d("Bitmap", "内存占用：${byteCount / 1024 / 1024} MB")
```

# 2 getByteCount和getAllocationByteCount的区别  

如果被复用的Bitmap的内存比待分配内存的Bitmap大  
getByteCount()获取到的是当前图片应当所占内存大小  
getAllocationByteCount()获取到的是被复用Bitmap真实占用内存大小  
在复用Bitmap的情况下，getAllocationByteCount()可能会比getByteCount()大  

# 3 Bitmap的压缩方式  

1. 质量压缩  

通过降低图片的质量精度（如JPEG的压缩率），牺牲画质来减小文件体积，但内存占用不变  

```java
fun compressQuality(bitmap: Bitmap, quality: Int, format: Bitmap.CompressFormat): ByteArray {
    val outputStream = ByteArrayOutputStream()
    bitmap.compress(format, quality, outputStream) // 质量参数（0-100）
    return outputStream.toByteArray()
}
```
适用于上传服务器，减少网络传输流量，保存到本地的图片  

注意：质量参数非线性，quality=50并不代表体积减半，如果是PNG这种无损格式，那么这个方法不会生效  

2. 尺寸压缩  

减少图片的像素数量，降低内存占用和文件体积  
```java
fun compressSize(bitmap: Bitmap, newWidth: Int, newHeight: Int): Bitmap {
    return Bitmap.createScaledBitmap(bitmap, newWidth, newHeight, true)
}
```

适用于对图片精度要求没有那么高的地方（如缩略图），大图加载避免OOM等  

3. 像素格式优化  

直接改变图片的像素格式  
```java
val options = BitmapFactory.Options().apply {
    inPreferredConfig = Bitmap.Config.RGB_565
}
val bitmap = BitmapFactory.decodeResource(resources, R.drawable.photo, options)
```

注意，改变像素格式可能导致显示问题，如透明背景变黑色不透明，慎用！  

4. 文件格式优化  

使用webp格式文件，有损模式比JPED小30%体积，无损格式比PNG小20%的体积  


# 4 LruCache和DiskLruCache原理  


# 5 如何设计一个图片加载库  


# 6 如果有一张很大的图片，如何去加载  



# 7 如果把drawable-xxhdpi下的图片移动到drawable-xhdpi下，图片内存如何变化  


# 8 如果hdpi、xxhdpi下都放置了图片，加载的优先级是怎样的，如果是400x800和1080x1920，加载的优先级  


# 9 屏幕分辨率适配方案  

