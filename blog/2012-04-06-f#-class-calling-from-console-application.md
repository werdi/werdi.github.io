---
title: C# beginner question - how to call F# class from Console Application
tags: [c#, f#]
src: https://social.msdn.microsoft.com/Forums/ru-RU/86ea747c-cc68-42cd-8d1f-eb61da6f39aa/c-beginner-question-how-to-call-f-class-from-console-application?forum=csharpgeneral
---
# C# beginner question - how to call F# class from Console Application
*> string loadWeb = new webClass(); [...] I can see webClass is in the reference, but how I can use it.*

instead of string loadWeb = new webClass(); 
write the following code:
```c#
var f = new cyber.fetch(null);
var res = f.getString("http://bing.com");
Console.Write(res); 
Console.ReadKey();
```

*> I don't understand why use " var f = new cyber.fetch(null);" I think var means the "f"  is an object, right?*

the var keyword instructs the compiler to infer the type of the variable.
so, instead of "var f = ..." you can write "cyber.fetch f = ..."
for more information, see var (C#)

*> To call class in F#, pass the parameter (null) to cyber.fetch?*

because the constructor's parameter is never used. also it seems the webString method is unnecessary.
so you can rewrite the fetch type as below:
```f#
type fetch() =
  member this.getString(url:string) = 
    use client = new WebClient()
    try
      client.DownloadString(url)
    with 
      | :? WebException -> ""
```
