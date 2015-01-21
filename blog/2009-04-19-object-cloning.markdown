---
title: Клонирование объектов
tags: [reflection, c#]
src: http://cognitex.blogspot.ru/2009/04/blog-post.html
---
# Клонирование объектов
Клонирование объектов, например, каких-нибудь простых DTO (data transfer object) или настроек можно выполнить с помощью MemberwiseClone:
```c#
public class Settings
{
    	public string Name { get; set; }
    	public int Value { get; set; }

    	public Settings CloneMemberwise()
    	{
        	return (Settings)this.MemberwiseClone();
    	}
}
```
По быстродействию это практически аналогично копированию свойств:
```c#
public Settings SimpleClone()
{
    	return new Settings() { Value = this.Value, Name = this.Name };
}
```
Но, бывает, что быстродействие некритично, класс невозможно наследовать (у класса определен модификатор sealed), а свойств много:
```c#
public sealed class Info
{
    	public string Prop1 { get; set; }
    	public string Prop20 { get; set; }
}
```
В таком случае клонировать можно с помощью reflection (отражение) и методов расширения:
```c#
public static class InfoHelper
{
    	public static Info Clone(this Info src)
    	{
        	Info ret = new Info();
        	foreach (var prop in src.GetType().GetProperties())
        	{
            	if (prop.CanWrite == false)
                	continue;

            	var val = prop.GetValue(src, null);
            	prop.SetValue(ret, val, null);
        	}
        	return ret;
    	}
}
```
Надо заметить, что этот способ раз в 6+ медленнее приведенных выше.
