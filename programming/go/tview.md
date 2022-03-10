

[TOC]

## 概念

### Pages

Pages is a container for other primitives often used as the application's root primitive. It allows to easily switch the visibility of the contained primitives. 它是 Box 的子类

### Table
- Table visualizes two-dimensional data consisting of rows and columns. Each Table cell is defined via SetCell() by the TableCell type. They can be added dynamically to the table and changed any time. The most compact display of a table is without borders. Each row will then occupy one row on screen and columns are separated by the rune defined via SetSeparator() (a space character by default). When borders are turned on (via SetBorders()), each table cell is surrounded by lines. Therefore one table row will require two rows on screen. Columns will use as much horizontal space as they need. You can constrain their size with the MaxWidth parameter of the TableCell type. 
- Fixed Columns: You can define fixed rows and rolumns via SetFixed(). They will always stay in their place, even when the table is scrolled. Fixed rows are always the top rows. Fixed columns are always the leftmost columns. 

- Selections: You can call SetSelectable() to set columns and/or rows to "selectable". If the flag is set only for columns, entire columns can be selected by the user.

- Navigation: If the table extends beyond the available space, it can be navigated with key bindings similar to Vim

### TextView

- 文本域

- TextView is a box which displays text. It implements the io.Writer interface so you can stream text to it. This does not trigger a redraw automatically but if a handler is installed via SetChangedFunc(), you can cause it to be redrawn. (See SetChangedFunc() for more details.)
- If the text view is scrollable (the default), text is kept in a buffer which may be larger than the screen and can be navigated similarly to Vim. 
- If the text is not scrollable, any text above the top visible line is discarded.
- Colors: If dynamic colors are enabled via SetDynamicColors(), text color can be changed dynamically by embedding color strings in square brackets. This works the same way as anywhere else. Please see the package documentation for more information.
- Regions and Highlights: 

## Layout

- Flex: 

## Box

### 方法

SetInputCapture：在一个 primitive  上安装一个事件捕获函数，在事件被发给 primitive 默认的事件处理之前捕获该事件，该函数可以选择将该事件或者另外一个事件发给默认的事件处理函数，也可以返回 nil，将该事件拦截掉。只有 focus 的 primitive 才能拦截事件。
