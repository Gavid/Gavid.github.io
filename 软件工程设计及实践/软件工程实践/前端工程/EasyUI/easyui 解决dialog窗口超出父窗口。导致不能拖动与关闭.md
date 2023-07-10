@[TOC](目录)
# 问题描述
> 今天做项目遇到父窗口调用diolog对话框时，出现diolog窗口的上边拖动和关闭部分超出了父窗口，导致不能拖动和关闭。
# 问题解决方法
> * 针对panel window dialog三个组件拖动时会超出父级元素的修正；
> * 如果父级元素的overflow属性为hidden，则修复上下左右个方向；
> * 如果父级元素的overflow属性为非hidden，则只修复上左两个方向；
# 问题解决实践
> 1. 将下方代码保存为 js 文件。
> 2. 将 js 文件在 html 文件中进行引用即可。
# 相关代码
```javascript
var easyuiPanelOnMove = function(left, top) {
var parentObj = $(this).panel('panel').parent();
if (left < 0) {
$(this).window('move', {
left : 1
});
}
if (top < 0) {
$(this).window('move', {
top : 1
});
}
var width = $(this).panel('options').width;
var height = $(this).panel('options').height;
var right = left + width;
var buttom = top + height;
var parentWidth = parentObj.width();
var parentHeight = parentObj.height();
if(parentObj.css("overflow")=="hidden"){
if(left > parentWidth-width){
$(this).window('move', {
"left":parentWidth-width
});
}
if(top > parentHeight-height){
$(this).window('move', {
"top":parentHeight-height
});
}
}
};
$.fn.panel.defaults.onMove = easyuiPanelOnMove;
$.fn.window.defaults.onMove = easyuiPanelOnMove;
$.fn.dialog.defaults.onMove = easyuiPanelOnMove;
```

参考文章链接 ：https://www.haorooms.com/post/jqueryeasyUI_dialog
