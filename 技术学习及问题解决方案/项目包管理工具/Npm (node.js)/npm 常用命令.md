# npm node.js包管理工具

> Note:
>
> 1. npm 命令需在 package.json 文件目录下执行。

## 常用命令整理：

```bash
# 更新安装包到指定版本
npm update packageName@版本号 
# 是将对应的安装包安装到当前项目的根目录下
npm install [packageName] 
# 查看安装包 packageName 最新发布的版本信息
npm view | info packageName version
# 查看全局安装的包
npm list -g --depth 0
# 查看当前目录的安装包
npm list
# 查看当前包源地址
npm config get registry
# 设置包源地址
npm config set registry http://ires.58corp.com/repository/58npm/
```

# 