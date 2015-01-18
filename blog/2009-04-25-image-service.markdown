# Сервис изображений
Например, необходимо на сайт добавить сервис, который будет динамически создавать изображения (.gif-формат) в ответ на запрос: "/image/logo,hello,RedOnMaroon", где logo - это тип изображения, hello - текст на изображении, Red - цвет текста, Maroon - цвет фона.
Ниже приводится метод, который необходимо добавить в [сервис] (http://cognitex.blogspot.com/2009/04/blog-post_24.html):
```c#
[OperationContract]
[WebGet(UriTemplate = "/image/{id},{text},{color}On{bgcolor}")]
public Stream GetImage(string id, string color, string bgcolor, string text)
{
    	var format = ImageFormat.Gif;

    	// создать
    	Bitmap img = new Bitmap(100, 100);
    	using (Graphics g = Graphics.FromImage(img))
    	{
        	g.Clear(Color.FromName(bgcolor));
        	g.DrawString(
            	text,
            	new Font("verdana", 14),
            	new SolidBrush(Color.FromName(color)),
            	new RectangleF(0, 0, img.Width, img.Height));
    	}

    	// сохранить
    	var ret = new MemoryStream();
    	img.Save(ret, format);
    	ret.Position = 0;

    	WebOperationContext.Current.OutgoingResponse.ContentType = ("image/" + format).ToLower();
    	return ret;
}
```
Пример вызова метода из html:
```html
<img src="/DataProvider.svc/image/logo,hello,WhiteOnRed" />
<img src="/DataProvider.svc/image/logo,привет,WhiteOnNavy" />
```
Если указать не все параметры, то клиент получит сообщение: Endpoint not found.

Если при отладке в методе GetImage вызвать img.Save("image." + format, format);, то файл будет создан в папке C:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE

В сервисе HttpContext.Current == null, но если: 

1) добавить в web.config в configuration следующий тег:
```xml
<system.serviceModel>
  	<serviceHostingEnvironment aspNetCompatibilityEnabled="true"/>
</system.serviceModel>
```
2) у класса сервиса определить атрибут AspNetCompatibilityRequirementsMode: 
```c#
[AspNetCompatibilityRequirements(RequirementsMode = AspNetCompatibilityRequirementsMode.Required)]
[ServiceContract]
public class DataProvider
{
    	...
```
3) в методе GetImage можно вызвать:
```c#
var dir = HttpContext.Current.Server.MapPath("~");
```
