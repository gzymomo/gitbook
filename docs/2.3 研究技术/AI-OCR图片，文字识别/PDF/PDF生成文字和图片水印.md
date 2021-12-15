[PDF生成文字和图片水印](https://www.cnblogs.com/wzh2010/p/13843233.html)

上传PDF文件到文件服务器上，都是一些合同或者技术评估文档，鉴于知识版权和防伪的目的，需要在上传的PDF文件打上水印，

这时候我们需要提供能力给客户，让他们可以对自己上传的文档，配置文字或者图片水印。

## 实现

于是我们参考了网上的一些资料，首选Spire.Pdf 和 iTextSharp，资料很多，是专业的PDF操作组件。

### Spire.Pdf

#### Spire Nuget安装

直接安装最新的版本就可以了

![点击查看大图](https://img2020.cnblogs.com/blog/167509/202010/167509-20201024204714729-815966185.png) 

#### Spire 代码段

这是生成图片水印，注释很清晰了。



```java
#region Spire.Pdf 组件

//创建PdfDocument对象
PdfDocument pdf = new PdfDocument();
//加载现有PDF文档
pdf.LoadFromFile(@"E:\WaterMark\ATAM.pdf");
//加载图片到System.Drawing.Image对象
System.Drawing.Image image = System.Drawing.Image.FromFile(@"E:\WaterMark\logo.png");
//遍历文档每一页(可以指定只是轮询某一个页面)
foreach (PdfPageBase page in pdf.Pages)
{
    //设置背景图
    page.BackgroundImage = image;
    //设置背景图的位置及大小（这边根据我们实际的图片大小进行同比缩小）
    page.BackgroundRegion = new RectangleF((page.ActualSize.Width - 500) / 2,
                                           (page.ActualSize.Height - 500)/2, 500, 250);
    //设置背景透明度
    page.BackgroudOpacity = 0.5f;
}
//保存并关闭文档（不关闭貌似打开的时候会有异常）
pdf.SaveToFile(@"E:\WaterMark\ATAM_WaterMark.pdf");
pdf.Close();

#endregion
```



#### Spire 实现结果

看着还行，起码达到效果了

![点击查看大图](https://img2020.cnblogs.com/blog/167509/202010/167509-20201024210210982-1447055469.png) 

#### Spire 扩展：文字水印 



```java
//加载PDF文档
PdfDocument pdf = new PdfDocument();
pdf.LoadFromFile(@"E:\WaterMark\ATAM.pdf");

//获取PDF文档的第一页
PdfPageBase page = pdf.Pages[0];

//绘制文本，设置文本格式并将其添加到页面
PdfTilingBrush brush = new PdfTilingBrush(new SizeF(page.Canvas.ClientSize.Width / 2, page.Canvas.ClientSize.Height / 3));
brush.Graphics.SetTransparency(0.3f);
brush.Graphics.Save();
brush.Graphics.TranslateTransform(brush.Size.Width / 2, brush.Size.Height / 2);
brush.Graphics.RotateTransform(-45);
PdfTrueTypeFont font = new PdfTrueTypeFont(new Font("Arial Unicode MS", 20f), true);
brush.Graphics.DrawString("这边定制你的文字水印", font, PdfBrushes.Red, 0, 0, new PdfStringFormat(PdfTextAlignment.Center));
brush.Graphics.Restore();
brush.Graphics.SetTransparency(1);
page.Canvas.DrawRectangle(brush, new RectangleF(new PointF(0, 0), page.Canvas.ClientSize));

//保存文档
pdf.SaveToFile(@"E:\WaterMark\ATAM_WaterMark.pdf");
pdf.Close();
```



### iTextSharp

#### iTextSharp Nuget安装

一样，直接安装最新版本即可

![点击查看大图](https://img2020.cnblogs.com/blog/167509/202010/167509-20201024211326869-1646654362.png) 

#### iTextSharp 代码段

这是iTextSharp 生成图片水印的方法代码，注释一样很清晰。



```java
/// <summary>
        /// 打水印功能
        /// </summary>
        /// <param name="infilepath">输入文件地址</param>
        /// <param name="outfilepath">输出文件地址</param>
        /// <param name="picName">图片文件地址</param>
        /// <param name="picHeight">图片高度（可选）</param>
        /// <param name="picWidth">图片宽度（可选）</param>
        /// <param name="top">图片在PDF中的位置Top（可选）</param>
        /// <param name="left">图片在PDF中的位置Left（可选）</param>
        /// <returns></returns>
        public bool PDFWatermark(string infilepath, string outfilepath, string picName,float picHeight=0,float picWidth=0,float top = 0,float left=0)
        {
            PdfReader pdfReader = null;
            PdfStamper pdfStamper = null;
            try
            {
                pdfReader = new PdfReader(infilepath);
                int numberOfPages = pdfReader.NumberOfPages;
                iTextSharp.text.Rectangle psize = pdfReader.GetPageSize(1);
                float width = psize.Width;
                float height = psize.Height;
                pdfStamper = new PdfStamper(pdfReader, new FileStream(outfilepath, FileMode.Create));
                PdfContentByte waterMarkContent;
                iTextSharp.text.Image image = iTextSharp.text.Image.GetInstance(picName);
                if (picHeight != 0) image.ScaleAbsoluteHeight(picHeight);
                else picHeight = image.Height;
                if (picWidth != 0) image.ScaleAbsoluteWidth(picWidth);
                else picWidth = image.Width;
                image.GrayFill = 20; // 透明度，灰色填充
                                     // image.Rotation // 旋转
                                     // image.RotationDegrees // 旋转角度
                // 水印的位置
                if (left == 0) left = (width - picWidth) / 2;
                if (top == 0) top = (height - picHeight) / 2;
                image.SetAbsolutePosition(left,top);
                // 每一页加水印,也可以设置某一页加水印
                for (int i = 1; i <= numberOfPages; i++)
                {
                    waterMarkContent = pdfStamper.GetUnderContent(i);//水印在最底层
                    //这边注意，如果想要水印在最顶层，这边改成 pdfStamper.GetOverContent
                    waterMarkContent.AddImage(image);
                }
                return true;
            }
            catch (Exception ex)
            {
                ex.Message.Trim();
                return false;
            }
            finally
            {

                if (pdfStamper != null) pdfStamper.Close();
                if (pdfReader != null) pdfReader.Close();
            }
        }
```



 应用



```java
#region iTextSharp
            string source = @"E:\WaterMark\ATAM.pdf"; //模板路径
            string output = @"E:\WaterMark\ATAM_WaterMark.pdf"; //导出水印背景后的PDF
            string watermark = @"E:\WaterMark\logo.png";   // 水印图片
            bool isSurrcess = PDFWatermark(source, output, watermark,250,500,0,0);
 #endregion
```



 

#### iTextSharp 实现结果

也不错，这个貌似实现起来更加灵活。

![点击查看大图](https://img2020.cnblogs.com/blog/167509/202010/167509-20201024212620478-1450733963.png)

#### iTextSharp 扩展：文字水印 



```java
/// <summary>
        /// 添加文字水印
        /// </summary>
        /// <param name="filePath">pdf文件地址</param>
        /// <param name="text">水印文本</param>
        public static void SetWatermark(string filePath, string text)
        {
            PdfReader pdfReader = null;
            PdfStamper pdfStamper = null;
            string tempPath = Path.GetDirectoryName(filePath) + Path.GetFileNameWithoutExtension(filePath) + "_temp.pdf";
            try
            {
                pdfReader = new PdfReader(filePath);
                pdfStamper = new PdfStamper(pdfReader, new FileStream(tempPath, FileMode.Create));
                int total = pdfReader.NumberOfPages + 1;
                iTextSharp.text.Rectangle psize = pdfReader.GetPageSize(1);
                float width = psize.Width;
                float height = psize.Height;
                PdfContentByte content;
                BaseFont font = BaseFont.CreateFont(@"C:\WINDOWS\Fonts\SIMFANG.TTF", BaseFont.IDENTITY_H, BaseFont.EMBEDDED);
                PdfGState gs = new PdfGState();
                for (int i = 1; i < total; i++)
                {
                    //在内容上方加水印（下方加水印参考上面图片代码做法）
                    content = pdfStamper.GetOverContent(i);
                   //透明度
                    gs.FillOpacity = 0.3f;
                    content.SetGState(gs);
                    //写入文本
                    content.BeginText();
                    content.SetColorFill(BaseColor.GRAY);
                    content.SetFontAndSize(font, 30);
                    content.SetTextMatrix(0, 0);
                    content.ShowTextAligned(Element.ALIGN_CENTER, text, width - 120, height - 120, -45);
                    content.EndText();
                }
            }
            catch (Exception ex)
            {
                throw ex;
            }
            finally
            {
                if (pdfStamper != null)pdfStamper.Close();
                if (pdfReader != null)pdfReader.Close();
                File.Copy(tempPath, filePath, true);
                File.Delete(tempPath);//删除临时文件
            }
        }
```

### 参考资料

Spire.Pdf：https://www.cnblogs.com/Yesi/p/4913603.html

iTextSharp：https://www.cnblogs.com/xishuqingchun/p/3838185.html