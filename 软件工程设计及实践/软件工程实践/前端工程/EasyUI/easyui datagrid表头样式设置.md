@[TOC]
# 需求描述
* 今天，接到一个项目需求，要求动态的将easyui中的datagrid中的表头样式进行修改。
	* 例如：将表头中的字体增大、将某个表头的字体加粗....
# 问题分析
* 通过查询网上资料，发现easyui并没有给出表头样式的动态设置，（可能自己的查询方式有误，希望有缘人能够查询出来的话，能够评论告知一下，万分感谢！）于是，只能通过easyui构建html代码的class属性，通过class属性找到对应元素，进行动态设置。
* 但是在实现过程中，发现easyui的部分class属性内有空格，使得通过class属性获取dom元素无法精准获取，于是，只能通过查找该dom元素的上层dom元素，然后通过遍历查询找到该dom元素进行样式的修改。
# 问题解决
```javascript
let headerText = ''; // 要修改的表头文本
let style = '';  // 要改成什么样式
$(".datagrid-header-row td div span").each(function(i,th){
	var val = $(th).text();
	if(val === headerText){
		$(th).html("<label style="+style+">"+val+"</label>");
	}
});

```
