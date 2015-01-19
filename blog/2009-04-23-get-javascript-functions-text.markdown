# Как получить текст JavaScript-функции
Если имя JavaScript-функции известно, и функция определена в одном из тегов SCRIPT на веб-странице, либо она находится в отдельном .js-файле, то ее текст можно получить в Silverlight-приложении следующим образом:
```javascript
var tmp = HtmlPage.Window.Eval("'' + MyFunc;");
```
Если ''+ не указать, то в tmp окажется ссылка на System.Windows.Browser.ScriptObject;
Если функция не определена, то при вызове метода Eval в IE8 всплывет диалог Webpage Error с вопросом: Do you want to debug this webpage? This webpage contains errors that might prevent it from displaying or working correctyly. If you are not testing this webpage, click No. 
После закрытия диалога в Silvelight-приложении получим System.InvalidOperationException.
Решение:
```c#
public static string GetFunctionCode(this HtmlWindow wnd, string fname)
{
    	return (string)wnd.Eval("(typeof " + fname + " == 'function') ? ('' + " + fname + ") : null;");
}
```
