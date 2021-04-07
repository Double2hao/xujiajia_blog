#android从相册中获取图片出错，图片无法裁剪的问题
笔者在学习获取相册中图片进行裁剪的时候遇到了比较大的问题，在纠结了近半天才真的解决，下面分享一下学习经验。

问题：

笔者在选择了相册中的图片之后要进入图片裁剪的时候出错，（华为）手机提示“此图片无法获取”，经百度后，明白是版本不同导致的URI的问题的问题，原文如下：

 

4.3或以下,选了图片之后,根据Uri来做处理,很多帖子都有了,我就不详细说了.主要是4.4,如果使用上面pick的原生方法来选图,返回的uri还是正常的,但如果用ACTION_GET_CONTENT的方法,返回的uri跟4.3是完全不一样的,4.3返回的是带文件路径的,而4.4返回的却是content://com.android.providers.media.documents/document/image:3951这样的,没有路径,只有图片编号的uri.这就导致接下来无法根据图片路径来裁剪的步骤了.

原文：

 

笔者在程序可以运行之后也是进行了一定的测试，如下图：

 

 

首先是用onActivityResult接收到的返回值作为Toast输出：

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1630.png">

 

得到如下效果：

 

<img alt="" class="has" height="700" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1631.png" width="400">

 

 

然后是用该图片的绝对路径作为Toast输出：

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1632.png">

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1633.png">

 

得到如下效果：

 

<img alt="" class="has" height="700" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1634.png" width="400">

 

 

果然是如该博文所讲，那么到底要如何修改呢？由于各个读者碰到的问题不同，笔者在此也无法说出一个万全的方法了，只能附上笔者从相册中选择图片并且裁剪的源码仅作参考。（其实知道了具体的问题之后，比较有经验的读者就可以自己解决了）

 

如实在有不懂的，可以访问笔者的上一篇博客，运行一下笔者提供的demo。

 

 

MainActivity：

 

```
import java.io.File;
import java.io.FileNotFoundException;
import java.io.IOException;

import android.app.Activity;
import android.content.Intent;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.net.Uri;
import android.os.Bundle;
import android.os.Environment;
import android.provider.MediaStore;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.Toast;

public class MainActivity extends Activity {

	public static final int CUT_PICTURE = 1;

	public static final int SHOW_PICTURE = 2;

	private Button takePhoto;

	private Button chooseFromAlbum;

	private ImageView picture;

	private Uri imageUri;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		takePhoto = (Button) findViewById(R.id.take_photo);
		chooseFromAlbum = (Button) findViewById(R.id.choose_from_album);
		picture = (ImageView) findViewById(R.id.picture);

		takePhoto.setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				//创建File对象，用于存储拍照后的图片
				//将此图片存储于SD卡的根目录下
				File outputImage = new File(Environment.getExternalStorageDirectory(),
						"output_image.jpg");
				try {
					if (outputImage.exists()) {
						outputImage.delete();
					}
					outputImage.createNewFile();
				} catch (IOException e) {
					e.printStackTrace();
				}
				//将File对象转换成Uri对象
				//Uri表标识着图片的地址
				imageUri = Uri.fromFile(outputImage);
				//隐式调用照相机程序
				Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
				//拍下的照片会被输出到output_image.jpg中去
				intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
				//此处是使用的startActivityForResult（）
				//因此在拍照完后悔有结果返回到onActivityResult（）中去，返回值即为TAKE_PHOTO
				//onActivityResult（）中主要是实现图片裁剪
				startActivityForResult(intent, CUT_PICTURE);
			}
		});

		chooseFromAlbum.setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				File outputImage = new File(Environment.getExternalStorageDirectory(),
						"output_image.jpg");
				imageUri = Uri.fromFile(outputImage);

				try {
					if (outputImage.exists()) {
						outputImage.delete();
					}
					outputImage.createNewFile();
				} catch (IOException e) {
					e.printStackTrace();
				}
				Intent intent = new Intent(Intent.ACTION_PICK,null);
				//此处调用了图片选择器
				//如果直接写intent.setDataAndType("image/*");
				//调用的是系统图库
				intent.setDataAndType(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, "image/*");
				intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
				startActivityForResult(intent, CUT_PICTURE);
			}
		});
	}


	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		switch (requestCode) {
		case CUT_PICTURE:
			if (resultCode == RESULT_OK) {
				//此处启动裁剪程序
				Intent intent = new Intent("com.android.camera.action.CROP");
				//此处注释掉的部分是针对android 4.4路径修改的一个测试
				//有兴趣的读者可以自己调试看看
				String text=imageUri.toString();
				Toast.makeText(MainActivity.this, text, Toast.LENGTH_SHORT).show();
				intent.setDataAndType(data.getData(), "image/*");
				intent.putExtra("scale", true);
				intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
				startActivityForResult(intent, SHOW_PICTURE);
			}
			break;
		case SHOW_PICTURE:
			if (resultCode == RESULT_OK) {
				try {
					//将output_image.jpg对象解析成Bitmap对象，然后设置到ImageView中显示出来
					Bitmap bitmap = BitmapFactory.decodeStream(getContentResolver()
							.openInputStream(imageUri));
					picture.setImageBitmap(bitmap);
				} catch (FileNotFoundException e) {
					e.printStackTrace();
				}
			}
			break;
		default:
			break;
		}
	}



}
```

 

 

 

 

 

 