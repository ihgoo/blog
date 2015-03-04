title: Tesseract-OCR 简单入门（一）
date: 2015-1-31 18:34:22
categories: Captcha
---
###介绍
Tesseract是一个开源的OCR引擎，遵循Apache2.0，是由HP实验室开始研发，96年因HP放弃ORC业务，Tesseract也从此停止开发。HP将其项目开源，2006年由谷歌重启该项目。

[Tesseract项目页](http://code.google.com/p/tesseract-ocr/)，他支持识别很多种语言（包括中文），非常容易使用，Tesseract项目本身没有GUI，只提供了命令行工具。

Tesseract支持通过训练来提高识别率，那么就让我们来体验一下吧
<!--more-->

###命令行参数说明

	Usage:tesseract imagename outputbase [-l lang] [-psm pagesegmode] [configfile...
	]

	pagesegmode values are:
	0 = Orientation and script detection (OSD) only.
	1 = Automatic page segmentation with OSD.
	2 = Automatic page segmentation, but no OSD, or OCR
	3 = Fully automatic page segmentation, but no OSD. (Default)
	4 = Assume a single column of text of variable sizes.
	5 = Assume a single uniform block of vertically aligned text.
	6 = Assume a single uniform block of text.
	7 = Treat the image as a single text line.
	8 = Treat the image as a single word.
	9 = Treat the image as a single word in a circle.
	10 = Treat the image as a single character.
	-l lang and/or -psm pagesegmode must occur before anyconfigfile.

	Single options:
	  -v --version: version info
	  --list-langs: list available languages for tesseract engine


	  tesseract imagename outputbase [-l lang] [-psm pagesegmode] [configfile
	  tesseract 需要识别的图片 输出路径名称 [-l 语言] [-psm 版面参数设置，详见上面参数说明]

###使用
核心代码如下所示
OCRHelper类，提供识别方法

	public class OCRHelper {
		private String tessPath = new File("tesseract").getAbsolutePath();

		public String Image2Text(File file, int times) throws IOException,
				InterruptedException {

			
			long start = System.currentTimeMillis();
			File outFile = new File(file.getParentFile(), times + "");

			List<String> command = new ArrayList();

			// tesseract imagename outputbase [-l lang] [-psm pagesegmode]
			// [configfile...
			command.add("tesseract");
			command.add(file.getName());
			command.add(outFile.getName());
			command.add("-psm");
			command.add("6");

			ProcessBuilder pBuilder = new ProcessBuilder();

			pBuilder.directory(file.getParentFile());
			pBuilder.redirectErrorStream(true);

			pBuilder.command(command);
			Process process = pBuilder.start();
			process.waitFor();
			

			BufferedReader br = new BufferedReader(new InputStreamReader(
					new FileInputStream(file.getParentFile() + "\\" + times + ".txt")));

			String result = br.readLine();
			long useTime = System.currentTimeMillis()-start;
			System.out.println("用时"+useTime+"毫秒");
			return StringUtils.filterBlack(result);
		}

	}


截取一段文字，以png格式保存到D盘下，执行命令，读取识别出来的验证码。

字符串去除空白类

	public class StringUtils {

		/**
		 * 去除空白
		 * @param content
		 * @return
		 */
		public static String filterBlack(String content){
			return content.replaceAll("\\s+","");
		}
	}

测试类


	public class Main {

		public static void main(String[] args) {

			OCRHelper ocrHelper = new OCRHelper();
			File file = new File("D:\\1.png");
			String image2Text = null;
			try {
				image2Text = ocrHelper.Image2Text(file, 0);
			} catch (IOException e) {
				e.printStackTrace();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

			System.out.println("识别结果为:" + image2Text);
		}

	}

**识别结果**

![](http://ihgoo.qiniudn.com/tesseract_resulttesseractResult.png)

对于规则的识别还是蛮不错的，再稍微复杂一点的就需要通过训练来增加识别率了。训练也有前人写了GUI的程序了，直接拿来用就好，下篇研究下训练后识别稍微复杂点儿的验证码。