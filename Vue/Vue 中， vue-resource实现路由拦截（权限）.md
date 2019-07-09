@[TOC](Vue 中， vue-resource实现路由拦截（权限）)
# 需求描述
> 在使用 Vue 做前端项目时，如果该用户尚未登录则将其重定向到登录页面进行登录，登录成功后才能浏览系统其他页面。
# 问题解决方式
> Vue 路由功能与 vue-resource 的拦截功能进行结合，即可结局本问题
# 问题具体解决过程

```javascript
import Vue from 'vue'
import Router from 'vue-router'
 
Vue.use(Router)
 
const router = new Router({
  routes: [
    {
      path: '/',
      /*
      *  按需加载 
      */
      component: (resolve) => {
        require(['../components/Home'], resolve)
      }
    }, {
      path: '/record',
      name: 'record',
      component: (resolve) => {
        require(['../components/Record'], resolve)
      }
    }, {
      path: '/Register',
      name: 'Register',
      component: (resolve) => {
        require(['../components/Register'], resolve)
      }
    }, {
      path: '/Luck',
      name: 'Luck', 
        // 需要登录才能进入的页面可以增加一个meta属性
      meta: { 
        requireAuth: true
      },
      component: (resolve) => {
        require(['../components/luck28/Luck'], resolve)
      },{
        path: '/vender', //厂商编码管理
        name: 'vender',
        meta: {
            requireAuth: true
        },
        component: resolve => require(['../components/content/venderCoding/venderCoding.vue'], resolve),
    },
    }
  ]
})
//  判断是否需要登录权限 以及是否登录
router.beforeEach((to, from, next) => {
  if (to.matched.some(res => res.meta.requireAuth)) {// 判断是否需要登录权限
    if (localStorage.getItem('username')) {// 判断是否登录
      next()  //有登录名称进行下一步路由
    } else {// 没登录则跳转到登录界面
      next({
        path: '/Register',
        query: {redirect: to.fullPath} //这一句我的项目中没有用到这一句
      })
    }
  } else {
    next()
  }
})
 
export default router

```

