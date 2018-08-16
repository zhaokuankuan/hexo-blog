---
title: java后台实现生成二维码并且上传的详细介绍
comments: true
tags: [二维码,java,Spring]
categories: [JavaWeb]
date: 2018-08-11 17:34:31
---
	 今天又遇到了新的问题，就是后台需要生成一个二维码，以前从来没有做过这个二维码，通过上午的努力，总算是完成了，希望有兴趣的可以一起交流学习。我用的是google.zxing的这个插件来完成生成二维码的，还是很方便的。
   一.首先需要引入google.zxing的jar包。
    我是建的maven工程，因此直接给你上maven的配置，别的可以在网上下载相应的jar包也行。pom.xml配置如下：
```
<!-- 二维码 -->
		<!-- https://mvnrepository.com/artifact/com.google.zxing/core -->
		<dependency>
		    <groupId>com.google.zxing</groupId>
		    <artifactId>core</artifactId>
		    <version>2.1</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.google.zxing/javase -->
		<dependency>
		    <groupId>com.google.zxing</groupId>
		    <artifactId>javase</artifactId>
		    <version>2.1</version>
		</dependency>
```
二.接下来直接上代码：
	        就是先将需要放在二维码中的数据传进来，生成二维码，然后在将二维码写进一个img中，最后是将这个图片放在了服务器上，然后这个二维码就完成了生成和上传，这次由于项目比较着急，没有时间去研究如何加水印，有兴趣的可以去研究下。
   具体的代码如下：
```
private static final int BLACK = 0xff000000;
	private static final int WHITE = 0xFFFFFFFF;

	/**
	 * @param args
	 */
	public static String tomakeMode(String strJson,String path) {
		QsMode test = new QsMode();
		String filePostfix="png";
		String UUID = StringUtil.getUUID();
  	    File file = new File(path  +UUID + "."+filePostfix);
		test.encode(strJson, file,filePostfix, BarcodeFormat.QR_CODE, 5000, 5000, null);
		return UUID+".png";
	}

	/**
	 *  生成QRCode二维码<br> 
	 *  在编码时需要将com.google.zxing.qrcode.encoder.Encoder.java中的<br>
	 *  static final String DEFAULT_BYTE_MODE_ENCODING = "ISO8859-1";<br>
	 *  修改为UTF-8，否则中文编译后解析不了<br>
	 * @param contents 二维码的内容
	 * @param file 二维码保存的路径，如：C://test_QR_CODE.png
	 * @param filePostfix 生成二维码图片的格式：png,jpeg,gif等格式
	 * @param format qrcode码的生成格式
	 * @param width 图片宽度
	 * @param height 图片高度
	 * @param hints
	 */
	public  void encode(String contents, File file,String filePostfix, BarcodeFormat format, int width, int height, Map<EncodeHintType, ?> hints) {
		try {
			contents = new String(contents.getBytes("UTF-8"), "ISO-8859-1"); 
			BitMatrix bitMatrix = new MultiFormatWriter().encode(contents, format, width, height);
			writeToFile(bitMatrix, filePostfix, file);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 生成二维码图片<br>
	 * 
	 * @param matrix
	 * @param format
	 *            图片格式
	 * @param file
	 *            生成二维码图片位置
	 * @throws IOException
	 */
	public  static void writeToFile(BitMatrix matrix, String format, File file) throws IOException {
		BufferedImage image = toBufferedImage(matrix);
		ImageIO.write(image, format, file);
	}

	/**
	 * 生成二维码内容<br>
	 * 
	 * @param matrix
	 * @return
	 */
	public static BufferedImage toBufferedImage(BitMatrix matrix) {
		int width = matrix.getWidth();
		int height = matrix.getHeight();
		BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_ARGB);
		for (int x = 0; x < width; x++) {
			for (int y = 0; y < height; y++) {
				image.setRGB(x, y, matrix.get(x, y) == true ? BLACK : WHITE);
			}
		}
		return image;
	}
```
    strJson是我向二维码中存入的数据，path是我的存放这个生成的二维码的路径，我是直接生成之后上传到服务器上，然后把下载的路径返回，以后需要的话可以直接下载。
    UUID是我的一个工具类，用来生成随机的编码的，主要是为了防止生成的二维码的名称重复，为了上传使用。
    上面的代码也是比较详细的，如果有不懂的地方可以一起讨论，相互学习，这个是原创，转载的话请注明来源。