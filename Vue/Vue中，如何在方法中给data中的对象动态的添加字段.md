@[toc]
# 1.问题描述：

今天，要将使用 Vue 前端框架编写的项目，想要在服务器进行项目的部署，由于Vue框架的 webpack 具有打包功能，但是在打包到部署过程中还存在一些问题，现在我将打包部署过程进行详细的描述一次，避免以后踩坑。

Question：可能会出现在浏览器浏览项目，出现空白页的情况。

# 2.问题解决办法：

通过查询资料，产生空白页的原因是由于项目的路径有问题。对项目打包时的路径进行修改，即可解决。

# 3.问题解决详细过程：

1.将webpack.prod.conf.js文件中的 module 中的 extract 属性值改为 false.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215162825912.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
2.打开 config 文件夹下的 index.js ，将 build 中的 assetsPublicPath 属性值 由 ‘ / ’ 修改为 ‘ ./ ’(斜杠前面加 点 )
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215163150415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
3.在 Terminal 控制窗口中，运行 “ npm run build ” , (稍等会)， 会下根目录下出现一个 dist 文件夹，如果服务器端的项目容器为 Apache 下的 Tomcat , 则只需要将 dist 文件夹（也可以将其进行重命名）放到Tomcat的webapps文件夹下，即部署完成，访问路径为 ( http://本机IP:8080/dist/ ) 
