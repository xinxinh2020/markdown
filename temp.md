mxgraph: 如何更新节点信息并渲染



shape: rectangle  ellipse doubleEllipse  rhombus line image arrow arrowConnector label cylinder swimlane connector actor triangle hexagon





```
strokeColor=blue;verticalAlign=bottom
```



```
style[mxConstants.STYLE_SHAPE] = mxConstants.SHAPE_ELLIPSE;
style[mxConstants.STYLE_PERIMETER] = mxPerimeter.EllipsePerimeter;
style[mxConstants.STYLE_FONTCOLOR] = 'white';
style[mxConstants.STYLE_GRADIENTCOLOR] = 'white';
style[mxConstants.STYLE_FONTSTYLE] = mxConstants.FONT_BOLD;
style[mxConstants.STYLE_FONTSIZE] = 14;
style[mxConstants.STYLE_SHADOW] = true;
```