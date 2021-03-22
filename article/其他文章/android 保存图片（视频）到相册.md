#android 保存图片（视频）到相册
# 概述

此功能麻烦的地方主要在机型适配上。 此功能实现步骤如下：
1. 将图片存储到手机picture目录下。（一般是从原位置复制过来）1. 将该文件扫描到相册。
>  
 本文的例子是将应用自带的内容存储到picture目录。 笔者看到目前市场上一些app有直接存储到DCIM目录下的。也是可行的，但是笔者个人认为最好还是只有相机相关功能生成的图片与视频存储到该目录下，否则就用户体验来讲可能会让人比较奇怪。 另外，不同目录在不同手机 ”相册“中的表现也有所区别，仅拿小米手机举例：picture目录中存储的内容可以再相册的”最近“中看到，而"DCIM/Camera"中存储的内容只能在相册的”相机“中看到。 


# 源码

```
public class ImageUtils {<!-- -->

  //在picture目录下新建一个自己文件夹
  private static final String rootPath =
      Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES) + "/appname";

  /**
   * 这个方法用来把已经存在的一个文件存储到相册
   * 
   * @param context 用来发送广播
   * @param srcString 需要拷贝的文件的地址
   */
  public static void saveFileToAlbum(Context context, String srcString) {<!-- -->
    if (TextUtils.isEmpty(srcString)) {<!-- -->
      return;
    }
    File srcFile = new File(srcString);
    if (!srcFile.exists()) {<!-- -->
      return;
    }
    //如果root文件夹没有需要新建一个
    createDirIfNotExist();

    //拷贝文件到picture目录下
    File destFile = new File(rootPath + "/" + srcFile.getName());
    copyFile(srcFile, destFile);

    //将该文件扫描到相册
    MediaScannerConnection.scanFile(context, new String[] {<!-- --> destFile.getPath() }, null, null);
  }

  public static void createDirIfNotExist() {<!-- -->
    File file = new File(rootPath);
    if (!file.exists()) {<!-- -->
      try {<!-- -->
        file.mkdirs();
      } catch (Exception e) {<!-- -->
        e.printStackTrace();
      }
    }
  }

  public static void copyFile(File src, File dest) {<!-- -->
    if (!src.getAbsolutePath().equals(dest.getAbsolutePath())) {<!-- -->
      try {<!-- -->
        InputStream in = new FileInputStream(src);
        FileOutputStream out = new FileOutputStream(dest);
        byte[] buf = new byte[1024];

        int len;
        while ((len = in.read(buf)) &gt;= 0) {<!-- -->
          out.write(buf, 0, len);
        }
        in.close();
        out.close();
      } catch (IOException e) {<!-- -->
        e.printStackTrace();
      }
    }
  }
}


```

# 发送广播扫描相册

笔者也使用过使用广播扫描相册的方式。 实际使用的时候会出现相册刷新不及时，在个别机型上没有刷新的问题，于是最终还是采用了上面的做法。

使用广播扫描相册可以扫描某个目录下的所有文件，而上面demo中的做法是只扫描某个文件。

```
    Intent intent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
    Uri uri = Uri.fromFile(new File(rootPath));
    intent.setData(uri);
    context.sendBroadcast(intent);

```

# 部分vivo手机的问题

部分vivo手机的视频只有存储在”相机“目录下的时候才会刷新生效。 笔者碰到这个问题的处理逻辑如下：
1. 先判断是否是vivo手机1. 如果是vivo手机，那么将视频在”相机“目录多拷贝一份，也多做一次刷新逻辑。