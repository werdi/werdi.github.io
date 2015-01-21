---
title: Работа с метаинформацией
tags: [attribute, c#]
src: http://cognitex.blogspot.ru/2009/07/enum.html
---
# Работа с метаинформацией
Метаинформация или метаданные - это информация об информации или другими словами - дополнительная информация. Например, есть следующее перечисление:
```c#
public enum Test
{
    	First,
    	Second,
    	Third
}
```
Требуется добавить метаданные к элементам перечисления. Сделать это можно с помощью атрибутов. 
Например, к каждому элементу можно добавить его описание с помощью атрибута Description:
```c#
public enum Test
{
    	[Description("First Element")]
    	First,
    	[Description("Second Element")]
    	Second,
    	[Description("Third Element")]
    	Third
}
```
Надо заметить, что Description - это краткая форма [DescriptionAttribute] (http://msdn.microsoft.com/ru-ru/library/system.componentmodel.descriptionattribute.aspx) (т.е. Attribute можно не указывать).
Для того, чтобы прочитать значение DescriptionAttribute.Description, который определен у Test.Second, надо вызвать метод GetDescription:
```c#
var description = Test.Second.GetDescription();
```
Метод является extension method'ом; далее приводится его реализация:
```c#
public static class EnumerationExtension
{
    	public static string GetDescription(this Enum value)
    	{
        	var da = GetAttribute<DescriptionAttribute>(value);
        	return da != null ? da.Description : string.Empty;
    	}

    	public static A GetAttribute<A>(this Enum value) where A : Attribute
    	{
        	var type = value.GetType();
        	var name = Enum.GetName(type, value);
        	var field = type.GetField(name);
        	return field.GetCustomAttributes(typeof(A), true).Cast<A>().FirstOrDefault();
    	}
}
```
P.S.
если требуется по Descriptor'у найти соответствующее значение перечисления, то надо вызвать метод FindValue:
```c#
Test res;
if (EnumExtention.FindValue<Test>("Second Element", out res))
{
    	// ...
}
```
Ниже приводится реализация метода:
```c#
public static class EnumExtention
{
    	public static bool FindValue<E>(string description, out E ret)
    	{
        	var type = typeof(E);
        	foreach (var item in Enum.GetNames(type))
        	{
            	var da = GetAttribute<DescriptionAttribute>(type, item);
            	if (da != null && string.Equals(da.Description, description, StringComparison.OrdinalIgnoreCase))
            	{
                	ret = (E)Enum.Parse(type, item);
                	return true;
            	}
        	}
        	ret = default(E);
        	return false;
    	}

    	public static A GetAttribute<A>(Type type, string fieldName)
    	{
        	var field = type.GetField(fieldName);
        	return field.GetCustomAttributes(typeof(A), true).Cast<A>().FirstOrDefault();
    	}
}
```
