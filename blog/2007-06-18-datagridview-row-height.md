---
title: Высота строки в DataGridView
tags: [datagridview, c#]
src: http://mindberg.blogspot.ru/2007/06/datagridview.html
---
# Высота строки в DataGridView
Высоту строк можно задавать в обработчике события DataGridView.RowHeightInfoNeeded. Например, если высота каждой второй строки должна быть равна 50 пикселей, то в обработчике пишем следующее:
```c#
if(e.RowIndex % 2 != 0)
  e.Height = 50;
else
  e.Height = 24;
```
else … – это обязательно; иначе высота у всех строк будет одинаковая.
