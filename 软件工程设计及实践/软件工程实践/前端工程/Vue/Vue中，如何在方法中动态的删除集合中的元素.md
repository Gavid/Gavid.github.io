1.问题描述：

>今天遇到一个问题，就是在前端想要动态更新集合数据（删除）的时候（初始数据全部信息已保存在集合中）

2.问题解决方式：

>通过查资料，集合存在动态删除元素的方法——splice()
>![在这里插入图片描述](https://img-blog.csdnimg.cn/20190227210019892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)

2.具体解决方式：

```javascript
		for(var k = 0;k< curThis.List.length;k++){
            if(curThis.List[k].Status === 3){
              curThis.List.splice(k, 1);
              k--;
            }
          }
```


