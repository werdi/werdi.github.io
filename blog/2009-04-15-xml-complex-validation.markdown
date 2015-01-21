---
title: Сложная валидация XML
tags: [xml, xmlreader, xpath, xsd, c#]
src: http://cognitex.blogspot.ru/2009/04/xml_15.html
---
# Сложная валидация XML
Если во время валидации xml требуется выполнять какие-то вычисления, запрашивать данные из БД, фиксировать Xpath к некорректному тегу и т.д. ... Решение: 
```c#
XmlElementReader reader = new XmlElementReader();
reader.ProcessElement += delegate
{
    	// для текущего элемента получить XPath и ...
    	Trace.WriteLine(reader.GetXPath());
};
reader.Read("test.xml");
```
Код XmlElementReader'а приводится ниже. Работает он следующим образом: с помощью XmlReader'а читает теги, их атрибуты, а также использует стек для фиксации пути к текущему тегу. После это посылает событие ProcessElement.
```c#
public class XmlElementReader
{
    	public event EventHandler ProcessElement = delegate { };

    	public XmlElementReader()
    	{
        	_Stack = new Stack<TagInfo>();
    	}
    	private Stack<TagInfo> _Stack;

    	public void Read(string xmlPath)
    	{
        	XmlReader r = XmlReader.Create(xmlPath);
        	while (r.Read())
        	{
            	if (r.NodeType == XmlNodeType.Element)
                	this.ReadElement(r);
        	}
    	}

    	public TagInfo[] GetPath()
    	{
        	return _Stack.ToArray().Reverse().ToArray();
    	}

    	[DebuggerDisplay("Name={Name}, Depth={Depth}, Index={Index}")]
    	public class TagInfo
    	{
        	public string Name { get; set; }
        	public int Index { get; set; }
        	public int Depth { get; set; }
        	public Dictionary<string, string> Attributes { get; set; }
    	}

    	private void ReadElement(XmlReader xr)
    	{
        	TagInfo stackTop = _Stack.Count > 0 ? _Stack.Peek() : null;
        	if (stackTop == null || xr.Depth > stackTop.Depth)
        	{
            	_Stack.Push(new TagInfo() { Name = xr.Name, Index = 0, Depth = xr.Depth, Attributes = xr.ReadAttributes() });
            	this.ProcessElement(this, EventArgs.Empty);
            	return;
        	}

        	if (xr.Depth < stackTop.Depth)
        	{
            	_Stack.Pop();
            	this.ReadElement(xr);    // рекурсия
            	return;
        	}

        	stackTop.Index++;
        	stackTop.Name = xr.Name;
        	stackTop.Attributes = xr.ReadAttributes();
        	this.ProcessElement(this, EventArgs.Empty);
    	}
}

public static class XmlReaderHelper
{
    	public static Dictionary<string, string> ReadAttributes(this XmlReader reader)
    	{
        	Dictionary<string, string> ret = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
        	if (reader.HasAttributes)
        	{
            	while (reader.MoveToNextAttribute())
                	ret.Add(reader.Name, reader.Value);

            	reader.MoveToElement();
        	}
        	return ret;
    	}
}

public static class XmlElementReaderHelper
{
    	public static string GetXPath(this XmlElementReader reader)
    	{
        	StringBuilder sb = new StringBuilder();
        	sb.Append("/");
        	foreach (var ti in reader.GetPath())
        	{
            	sb.Append(ti.Name);
            	string id;
            	if (ti.Attributes.TryGetValue("id", out id))
                	sb.Append("[@id='").Append(ti.Attributes["id"]).Append("']");

            	sb.Append("/");
        	}
        	return sb.ToString().TrimEnd('/');
    	}
}
```
