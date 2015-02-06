---
title: Свойство Path для aspx страницы. Маршрутизация.
tags: [c#, web]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b8ea42c2-4a17-4414-ba5a-61312c138494/-path-aspx-?forum=aspnetru
---
# Свойство Path для aspx страницы. Маршрутизация.
*> Чтобы к aspx странице можно было обращаться из любого места, т.е. http://localhost/myApp/1/1/1/bla.bla, а физическое размещение этой страницы находится в корне приложения: http://localhost/myApp/bla.aspx*

см. [How to: Use Routing with Web Forms] (http://msdn.microsoft.com/en-us/library/cc668202(v=vs.90).aspx)
и [Creating Rewrite Rules for the URL Rewrite Module] (http://learn.iis.net/page.aspx/461/creating-rewrite-rules-for-the-url-rewrite-module/)

*> Хочется просто добавить aspx страницу в любой проект и всяких телодвижений она заработает как указано в первом сообщении.*

как вариант можно определить свой VirtualPathProvider и возвращать любой text/html.
для этого надо создать папку App_Code и добавить в нее test.cs

```c#
using System; 
using System.Web; 
using System.Web.Util; 
using System.Web.Hosting; 
using System.IO;
using System.Text;

namespace WebApplication1
{
  public class PathProvider : VirtualPathProvider 
  {
    public static void AppInitialize()
    {
      HostingEnvironment.RegisterVirtualPathProvider(new PathProvider()); 
    }
    public override VirtualFile GetFile(string virtualPath)
    {
      return virtualPath.EndsWith("/bla.bla", StringComparison.OrdinalIgnoreCase)
        ? new File(virtualPath)
        : base.GetFile(virtualPath);
    }
    public override bool FileExists(string virtualPath)
    {
      return virtualPath.EndsWith("/bla.bla", StringComparison.OrdinalIgnoreCase)
        ? true
        : base.FileExists(virtualPath);
    }
  }
  
  class File : VirtualFile
  {
    public File(string vp)
      : base(vp)
    {
    }
    public override Stream Open()
    {
      var ms = new MemoryStream(Encoding.Default.GetBytes("bla.bla"));
      return ms;
    }
  }
}
```
