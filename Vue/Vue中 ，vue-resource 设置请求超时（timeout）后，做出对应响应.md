@[TOC](vue-resource 设置请求超时后，做出对应响应)
# 需求描述

> 在 HTTP 的请求时，若远程网络失去响应则会返回超时响应，但是前端会一直显示等待状态，现在想将在收到超时响应后，前端也进行一定的提示

# 问题解决方式
> 通过查询资料，了解到 vue-resource 组件中的==interceptors==已经提供了这种问题的解决方式。（timeout）

# 具体解决过程
## 1. vue-resource 中 inteceptor 的基本介绍
 拦截器可以在请求发送前和发送请求后做一些处理。下面是 inteceptor 的请求拦截方式：
 ![inteceptor 的请求拦截方式](https://img-blog.csdnimg.cn/20190319160651774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
 ## 2. inteceptor 的基本用法
 

```javascript
Vue.http.interceptors.push((request, next) => {
		// ...
		// 请求发送前的处理逻辑
		// ...
	next((response) => {
		// ...
		// 请求发送后的处理逻辑
		// ...
		// 根据请求的状态，response参数会返回给successCallback或errorCallback
		return response
	})
})
```
## 3. 使用 inteceptor 解决本问题
### 3.1 超时之后可以在then的第二个error方法中获取（==推荐==）
#### 3.1.1 在 main.js 中进行全局的请求拦截方式

```javascript
Vue.http.interceptors.push((request, next) => {
    let timeout;
    // 这里改用 _timeout
    if (request._timeout) {
        timeout = setTimeout(() => {
　　　　　　　　//自定义响应体 status:408,statustext:"请求超时"，并返回给下下边的next
            next(request.respondWith(request.body, {
                 status: 408,
                 statusText: '请求超时'
            }));
            
        }, request._timeout);
    }
    next((response) => {
　　　　console.log(response.status)//如果超时输出408
　　　　return response;
    })
})
```
#### 3.1.2 在 页面请求设置 中进行全局的请求拦截方式

```javascript
this.$http.get(`repairs/${this.repairs_id}`,{
                params:{with:'room;user'},
                _timeout:100,//设置超时时间
            }).then((response)=>{
            },(err)=>{
                console.log(err.status);//如果超时，此处输出408
});
```
### 3.2 超时之后会调用请求中的onTimeoutd方法，then方法不会执行
#### 3.2.1 在 main.js 中进行全局的请求拦截方式

```javascript
Vue.http.interceptors.push((request, next) => {
    let timeout;
    // 如果某个请求设置了_timeout,那么超过该时间，会终端该（abort）请求,并执行请求设置的钩子函数onTimeout方法，不会执行then方法。
    if (request._timeout) {
        timeout = setTimeout(() => {
            if(request.onTimeout) {
                request.onTimeout(request);
                request.abort()
            }  
        }, request._timeout);
    }
    next((response) => {
       clearTimeout(timeout);
　　　　return response;
    })
})
```

#### 3.2.2 在 页面请求设置 中进行全局的请求拦截方式

```javascript
this.$http.get('url',{
            params:{.......},
　　　　　　　......
           _timeout:3000,
           onTimeout: (request) => {
               alert("请求超时");
           }
     }).then((response)=>{
               
});
```

