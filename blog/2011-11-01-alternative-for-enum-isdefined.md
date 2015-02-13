---
title: Alternative for Enum.IsDefined
tags: [c#, winforms, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9e228871-fefd-4d2a-9f7d-f0be832447d6/alternative-for-enumisdefined?forum=csharpgeneral
---
# Alternative for Enum.IsDefined
*> I am just thinking if there is any other method or some extension method which will do it with less memory consumption and in quick time*

if i understood correctly you can use dictionary
```c#
var tbl = EnumHelper.ToDictionary<HeaderHistoryStatusEnum>();
if(tbl.ContainsKey("s"))
{
}
...
public static class EnumHelper
{
  public static IDictionary<string, T> ToDictionary<T>()
  {
    var t = typeof(T);
    if (t.IsEnum == false)
      throw new ArgumentException("IsEnum == false");
    var res = Enum.GetValues(t).Cast<T>().Select(v => new { Key = v.ToString(), Value = v });
    var tbl = new Dictionary<string, T>(StringComparer.OrdinalIgnoreCase);
    foreach (var p in res)
      tbl.Add(p.Key, p.Value);
    return tbl;
  }
}
```
*> I have a about a 100,000 items list.Each item is a string with certain possible values which I have listed in enums [...] Now in the current code I am validating it via help of Enum.Isdefined*

another approach is based on the linq
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Windows.Forms;
...
enum Test
{
  a,
  b,
}
...
var enums = Enum.GetValues(typeof(Test)).OfType<Test>().Select(v => v.ToString());
var list = new List<string> { "a", "b", "c", "d" };
var dif = list.Except(enums);
// result: "c", "d"
foreach(var itm in dif)
  System.Diagnostics.Trace.WriteLine(itm);
```
P.S.
sometimes, when you have a lot of data, it makes sense to load them into the database and then perform a comparing using sql in stored procedure

*> Can you please also let me see how you have cache these lists.*

do you asking about EnumHelper.ToDictionary method, mentioned above or some other?

*> I just wanted to see where is caching is take place.*

take a look at the code above. note the following
```c# 
var tbl = EnumHelper.ToDictionary<HeaderHistoryStatusEnum>();
``` 
tbl contains a cached values. 
please, note that no need to use ToDictionary more then once, if the tbl created. 
by the way, if you need to store tbl between some method calls, then store tbl into the [MemoryCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx).
*> Problem was infact not enum.Isdefine but one another method which check the a description over enumeration value ...*

I rethought your issue. it would be better to use just a static variables for caching instead of MemoryCache.
try using the following class.
```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
...
public static class EnumHelper
{
  static Dictionary<string, Enum> Values;
  static Dictionary<string, Enum> Descriptions;
  static Type GetType<T>()
  {
    var t = typeof(T);
    if (t.IsEnum == false) 
      throw new ArgumentException("<T>");
    return t;
  }
  static Dictionary<string, Enum> Create(Type et, Func<Enum, string> tostr)
  {
    var vals = Enum.GetValues(et).Cast<Enum>();
    var res = new Dictionary<string, Enum>(StringComparer.OrdinalIgnoreCase);
    foreach(var val in vals)
    {
      var str = tostr(val);
      if(str != null)
        res.Add(str, val);
    }
    return res;
  }
  public static bool HasDescription<T>(string desc)
  {
    return FindByDescription<T>(desc) != null;
  }
  public static Enum FindByDescription<T>(string desc)
  {
    var et = GetType<T>();
    if (Descriptions == null) 
      Descriptions = Create(et, v => 
        {
          var mi = et.GetMember(v.ToString())[0];
          var da = Attribute.GetCustomAttribute(mi, typeof(DescriptionAttribute)) as DescriptionAttribute;
          return da != null ? da.Description : null;
        });
    Enum res;
    Descriptions.TryGetValue(desc, out res);
    return res;
  }
  public static bool HasValue<T>(string value)
  {
    var et = GetType<T>();
    if (Values == null)
      Values = Create(et, v => v.ToString());
    return Values.ContainsKey(value);
  }
}
```
*> Then how to use it ...*

let's suppose you have the folowing enum
``` 
enum MyEnum
{
  [Description("desc a")]
  A,
  B
}
``` 
you can write something like this
```
if(EnumHelper.HasValue<MyEnum>("A") == false){/*...*/}
if(EnumHelper.HasDescription<MyEnum>("desc a") == false){/*...*/} 
```