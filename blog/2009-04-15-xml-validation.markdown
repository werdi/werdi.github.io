---
title: Валидация XML
tags: [xml, xmlreader, xsd]
src: http://cognitex.blogspot.ru/2009/04/xml.html
---
# Валидация XML
Например, есть следующий XML:
```xml
<?xml version="1.0" encoding="utf-8" ?>
<root>
  	<item id="0" />
  	<item id="1">
    	<item id="0" />
  	</item>
  	<item id="0" />
</root>
```
Требуется выполнить валидацию XML, например, проверить уникальность значений атрибутов id у вложенных (nested) тегов item. Для это надо определить схему xml в виде .xsd-файла:
```xml
<?xml version="1.0" standalone="yes"?>
<xs:schema id="root" xmlns="" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata">
  	<!-- корневым тегом может быть root -->
  	<xs:element name="root" msdata:IsDataSet="true" msdata:Locale="en-US">
    	<xs:complexType>
      	<!-- root может содержать неограниченное кол-во item -->
      	<xs:choice minOccurs="0" maxOccurs="unbounded">
        	<xs:element ref="item" />
      	</xs:choice>
    	</xs:complexType>
    	<!-- правило для атрибута id в item -->
    	<xs:unique name="uid" msdata:PrimaryKey="true">
      	<xs:selector xpath=".//item" />
      	<xs:field xpath="@id" />
    	</xs:unique>
  	</xs:element>
  	<!-- определение тега item -->
  	<xs:element name="item">
    	<xs:complexType>
      	<xs:sequence>
        	<!-- может содержать неограниченное кол-во подтегов item -->
        	<xs:element ref="item" minOccurs="0" maxOccurs="unbounded" />
      	</xs:sequence>
      	<!-- определение атрибута id -->
      	<xs:attribute name="id" type="xs:string" />
    	</xs:complexType>
  	</xs:element>
</xs:schema>
```
Валидация выполняется с помощью следующего кода:
```c#
XmlReaderSettings xs = new XmlReaderSettings();
xs.Schemas.Add("", new XmlTextReader(@"test.xsd"));
xs.ValidationType = ValidationType.Schema;
xs.ValidationFlags |= XmlSchemaValidationFlags.ReportValidationWarnings | XmlSchemaValidationFlags.ProcessIdentityConstraints;
xs.ValidationEventHandler += (s, e) =>
    	System.Diagnostics.Trace.WriteLine(e.Exception.LineNumber + " " + e.Message);

XmlReader xr = XmlReader.Create("test.xml", xs);
while (xr.Read()) ;
```
Информация обо всех ошибках передается в обработчик события ValidationEventHandler.
Для приведенного выше xml получим следующие сообщения: 
5 There is a duplicate key sequence '0' for the 'uid' key or unique identity constraint.
7 There is a duplicate key sequence '0' for the 'uid' key or unique identity constraint.
(5 и 7 - номера строк в xml)
