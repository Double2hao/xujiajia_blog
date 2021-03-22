#微信分享报错 checkArgs fail, thumbData is invalid （bitmap压缩）
# 概述

近期使用微信分享碰到报错：

>  
 checkArgs fail, thumbData is invalid 


查阅文档后发现是因为分享缩略图的大小不可超过32kb。 于是在框架里加入了压缩图片的逻辑。

# 源码

压缩逻辑很简单：查看数据是否超过限制，如果超过就将bitmap质量再减一半。

```
    /**
     * Bitmap转换成byte[]并且进行压缩,压缩到不大于maxkb
     * @param bitmap
     * @param maxkb
     * @return
     */
    public static byte[] bmpToByteArray(Bitmap bitmap, final boolean needRecycle, int maxkb) {<!-- -->
        if (bitmap == null) {<!-- -->
            return null;
        }
        try {<!-- -->
            ByteArrayOutputStream output = new ByteArrayOutputStream();
            byte[] result = null;
            bitmap.compress(Bitmap.CompressFormat.PNG, 100, output);
            int options = 100;
            while (output.toByteArray().length &gt; maxkb) {<!-- -->
                output.reset(); //清空output
                bitmap.compress(Bitmap.CompressFormat.JPEG, options, output);
                options /= 2;
            }
            result = output.toByteArray();
            if (needRecycle) {<!-- -->
                bitmap.recycle();
            }
            output.close();
            return result;
        } catch (Exception e) {<!-- -->
            e.printStackTrace();
        } 
        return null;
    }

```