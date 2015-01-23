---
title: Использование JSON в Silverlight 2.0
tags: [json, silverlight, c#]
src: http://mindberg.blogspot.ru/2008/10/json-silverlight-20_29.html
---
# Использование JSON в Silverlight 2.0
Для сериализации объектов в JSON предназначен DataContractJsonSerializer. 
Например, есть следующий класс:
```c#
public class DTO 
{ 
  public Guid Id { get; set; } 
  public string Value { get; set; } 
  public override string ToString() 
  { 
    return string.Concat("{Id='", this.Id, "', Value='", this.Value, "'}"); 
  } 
}
```
создаем экземпляр класса:
```c#
var ct = new DTO() { Id = Guid.NewGuid(), Value = "hello" }; 
```
сериализовать: 
```c#
string json = JSON.Serialize(ct); 
```
получим:
```json
{"Id":"a90417e2-9689-4d80-8c34-b1faeb14823b","Value":"hello"}
```
десериализовать:
```c#
DTO dto = JSON.Deserialize<DTO>(json);
--
public static class JSON 
{ 
  public static string Serialize(object value) 
  { 
    // создать сериализатор
    DataContractJsonSerializer js = new DataContractJsonSerializer(value.GetType());
    // создать стрим; и записать в него результат сериализации 
    MemoryStream ms = new MemoryStream(); 
    js.WriteObject(ms, value);
    // получить строку из стрима 
    // v1: 
    ms.Position = 0; 
    using (StreamReader reader = new StreamReader(ms)) 
      return reader.ReadToEnd(); 
      // v2: 
      // byte[] bytes = ms.ToArray(); 
      // return System.Text.Encoding.UTF8.GetString(bytes, 0, bytes.Length); 
  }
  public static T Deserialize<T>(string json) 
  { 
    // создать сериализатор 
    DataContractJsonSerializer js = new DataContractJsonSerializer(typeof(T)); 
    // создать стрим; и записать в него строку 
    MemoryStream ms = new MemoryStream(System.Text.Encoding.UTF8.GetBytes(json)); 
    // десериализовать ... 
    return (T)js.ReadObject(ms); 
  } 
}
```
