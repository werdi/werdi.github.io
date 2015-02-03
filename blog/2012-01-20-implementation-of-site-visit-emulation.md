---
title: C# или VB.Net. Программная реализация эмуляции посещения сайта
tags: [c#, winforms, webbrowser, automation]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d1eabb58-3c06-4225-91c1-331375845cbd/c-vbnet-?forum=programminglanguageru
---
# C# или VB.Net. Программная реализация эмуляции посещения сайта
*> нужно обновляться (или совершать другие манипуляции со страницей), что бы выглядело, что постоянно онлайн.*

загрузите страницу в WebBrowser. создайте таймер, и по таймеру обновляйте страницу.
если надо программно заполнять поля ввода и нажимать кнопки, то см. [здесь] (http://social.msdn.microsoft.com/Forums/en-US/csharpgeneral/thread/39d18e30-d300-407b-8f54-82d87a926e81/#967dad4e-6885-42f2-9af8-14dc42de906f)

*> на странице есть форма: [...] мне надо, что бы поля для логина и пароля были заполнены и нажалась после этого кнопка «Вход».*
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public ProgressBar ProgressBar;

    public Form1()
    {
      this.Size = new System.Drawing.Size(550, 450);

      this.Size = new System.Drawing.Size(800, 500);
      var wb = new WebBrowser { Parent = this, Dock = DockStyle.Fill, ScriptErrorsSuppressed = true };
      wb.DocumentCompleted += (s, e) =>
      {
        if (wb.ReadyState != WebBrowserReadyState.Complete)
          return;

        var u = wb.Document.GetElementById("user");
        u.Focus();
        this.Activate();
        SendKeys.SendWait("user");
        SendKeys.SendWait("{TAB}");
        SendKeys.SendWait("pass");
        SendKeys.SendWait("{ENTER}");
      };
      wb.Navigate("<адрес сайта>");
    }
  }
}
```
*> где хранится информация об авторизации в созданном браузере?*

обычно в [Cookie] (http://msdn.microsoft.com/en-us/library/system.windows.forms.htmldocument.cookie.aspx).
```c#
var wb = new WebBrowser {Parent = this, Dock = DockStyle.Fill };
wb.DocumentCompleted += (s, e) =>
{
  if(wb.ReadyState != WebBrowserReadyState.Complete) return;
  var str = wb.Document.Cookie;
  ...
};
wb.Navigate("http://microsoft.com");
```
*> Можно как-то эти данные использовать для входа на сайт. [...] Т.е. не использовать всю процедуру авторизации, а подключить эту строку для входа (авторизации). Это возможно?*

да, но возможно, что сайт с помощью проверочного javascript отслеживает ситуацию, когда запросы идут не из браузера. если не отслеживает, то для обмена данными с сайтом можно использовать [WebClient] (http://msdn.microsoft.com/ru-ru/library/system.net.webclient.aspx) или [HttpWebRequest] (http://msdn.microsoft.com/ru-ru/library/system.net.httpwebrequest.aspx)
чтобы понять какие данные пересылать, надо поставить [Fiddler] (http://www.fiddler2.com/fiddler2/) и отследить трафик и воспроизвести его из своего приложения.
