---
title: How to pass external value to xsl file?
tags: [html, xslt, hta, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/578fd6d1-18dc-4bc2-9e64-e28b54b49ecc/how-to-pass-external-value-to-xsl-file?forum=xmlandnetfx 
---
# How to pass external value to xsl file?
*> Below is my TEST.XML file [...]  TEST.XSL file [...] I need to pass a variable value to the xsl file how?*

use Msxml2.XSLTemplate addParameter function and the xsl:param tag.
below is an example

[Test.hta]
```html
<!doctype html>
<html>
<head>
  <title></title>
  <hta:application id="ohta" maximizebutton="no" minimizebutton="yes" innerborder="no"
    showintaskbar="yes" singleinstance="yes" scroll="no" version="1.0" />
  <style type="text/css">
    html, body { height: 100%; margin:0px; padding:0px; }
  </style>
  <script type="text/javascript">
    window.resizeTo((screen.width*0.60), (screen.height*0.50));
    function show(str) 
    {
      document.getElementById("res").innerHTML = str;
    }
    function loadXml(path)
    {
      var xml = new ActiveXObject("Msxml2.FreeThreadedDOMDocument");
      xml.async = false;
      xml.load(path);
      if(xml.parseError.errorCode != 0)
        show(xml.parseError.reason);
      return xml;
    }
    function createProcessor(xml, xsl)
    {
      var xslt = new ActiveXObject("Msxml2.XSLTemplate");
      xslt.stylesheet = xsl;
      var xslp = xslt.createProcessor();
      xslp.input = xml;
      return xslp;
    }
    window.onload = function() 
    {
      var xml = loadXml("Test.xml");
      var xsl = loadXml("Test.xslt");
      var xslp = createProcessor(xml, xsl);
      xslp.addParameter("test", "hello");
      xslp.transform();
      show(xslp.output);
    }
  </script>
</head>
<body>
  <pre id="res" />
</body>
</html>
```
[Test.xml]
```xml
<?xml version="1.0" encoding="utf-8" ?>
<root />
```
[Test.xslt]
```xslt
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
  xmlns:msxsl="urn:schemas-microsoft-com:xslt" exclude-result-prefixes="msxsl">
  <xsl:output method="xml" indent="yes"/>
  <xsl:param name="test" />
  <xsl:template match="/">
  	param: <xsl:value-of select="$test"/>
	</xsl:template>
</xsl:stylesheet>
```
also you can use external function in the xsl.
in the code above add the following lines:

[Test.hta]
```hta
...
xslp.addObject({ test:function() { return "ok " + new Date(); } }, "urn:my-object");
```
[Test.xslt]
```xslt
...
xmlns:cb="urn:my-object"
...
callback: <xsl:value-of select="cb:test(.)"/>
```
