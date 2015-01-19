---
title: Особенность CookieContainer'а
tags: [feature, webclient]
src: http://cognitex.blogspot.ru/2009/04/cookiecontainer.html
---
# Особенность CookieContainer'а
При первом обращении к веб-сайту, например, test.ru был получен ответ:
```
HTTP/1.1 200 OK
Date: Thu, 16 Apr 2009 08:39:02 GMT
Set-Cookie: PathDotTest=value; expires=Tue, 13-Apr-2010 23:43:04 GMT; path=/; domain=.test.ru
Set-Cookie: PathTest=value; expires=Tue, 13-Apr-2010 23:43:04 GMT; path=/; domain=test.ru
Set-Cookie: DotTest=value; expires=Tue, 13-Apr-2010 23:43:04 GMT; domain=.test.ru
Set-Cookie: WwwDotTest=value; expires=Tue, 13-Apr-2010 23:43:04 GMT; domain=www.test.ru
Set-Cookie: Test=value; expires=Tue, 13-Apr-2010 23:43:04 GMT; domain=test.ru
Set-Cookie: Path=value; expires=Tue, 13-Apr-2010 23:43:04 GMT; path=/;
Set-Cookie: Null=value; expires=Tue, 13-Apr-2010 23:43:04 GMT;
Set-Cookie: External=value; expires=Tue, 13-Apr-2010 23:43:04 GMT; domain=external.ru
Location: /homepage
Content-Length: 0
```
В HttpWebResponse.Cookies, а соответственно и в HttpWebRequest.CookieContainer окажется меньше cookie, чем указано в http-заголовках:
```
PathDotTest=value; expires=14.04.2010 3:43:04; path=/; domain=.test.ru
PathTest=value; expires=14.04.2010 3:43:04; path=/; domain=test.ru
DotTest=value; expires=14.04.2010 3:43:04; path=/; domain=.test.ru
Test=value; expires=14.04.2010 3:43:04; path=/; domain=test.ru
Path=value; expires=14.04.2010 3:43:04; path=/; domain=test.ru
Null=value; expires=14.04.2010 3:43:04; path=/; domain=test.ru
```
Два cookie: WwwDotTest и External - "пропали", т.е. не попали в контейнер. С External - понятно, это сторонний домен. А при повторном обращении к сайту:
    <ul>
      <li>адрес: http://test.ru; отправлены cookie: Path и Null</li>
      <li>адрес: http://www.test.ru; отправлены cookie: PathDotTest, PathTest, DotTest, Test</li>
    </ul>
Вот такая особенность есть у CookieContainer'а, которая противоречит правилам сайтов:
    <ul>
      <li>если в cookie указан domain=test.ru, то верни cookie при запросе http://test.ru</li>
      <li>если в cookie указан domain=mail.test.ru, то верни cookie при запросе http://mail.test.ru</li>
      <li>если в cookie указан domain=.test.ru, то верни cookie при запросе http://test.ru и любого поддомена, например, http://www.test.ru или http://mail.test.ru</li>
      <li>если domain не указан, то это эквивалентно http://test.ru</li>
    </ul>
        Т.е. иногда лучше самому обрабатывать http-заголовки.
    
