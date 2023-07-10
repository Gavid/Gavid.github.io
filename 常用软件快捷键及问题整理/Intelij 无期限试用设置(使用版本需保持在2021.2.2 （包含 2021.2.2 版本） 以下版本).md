## Intelij 无期限试用设置(使用版本需保持在2021.2.2 （包含 2021.2.2 版本） 以下版本)

### 1.安装插件

> 插件市场安装：
> 在Settings/Preferences… -> Plugins 内手动添加第三方插件仓库地址：[https://plugins.zhile.io](https://link.zhihu.com/?target=https%3A//plugins.zhile.io)
> 搜索：IDE Eval Reset插件进行安装。如果搜索不到请注意是否做好了上一步？网络是否通畅？
> 插件会提示安装成功。
>
> 下载安装：
> 下载插件的zip包 **（已下载到本目录下）**（macOS可能会自动解压，然后把zip包丢进回收站）
> 通常可以直接把zip包拖进IDE的窗口来进行插件的安装。如果无法拖动安装，你可以在Settings/Preferences… -> Plugins 里手动安装插件（Install Plugin From Disk…）
> 插件会提示安装成功。

### 2.使用

> 如果IDE没有打开项目，在Welcome界面点击菜单：Get Help -> Eval Reset
> 如果IDE打开了项目，点击菜单：Help -> Eval Reset
> 唤出的插件主界面中包含了一些显示信息，2个按钮，1个勾选项：
> 按钮：Reload 用来刷新界面上的显示信息。
> 按钮：Reset 点击会询问是否重置试用信息并重启IDE。选择Yes则执行重置操作并重启IDE生效，选择No则什么也不做。（此为手动重置方式）
> 勾选项**（推荐）**：Auto reset before per restart 如果勾选了，则自勾选后每次重启/退出IDE时会自动重置试用信息，你无需做额外的事情。（此为自动重置方式）

### 3.手动删除试用授权文件

有时，因为隔了太长一段时间没有打开过 ide，*eval reset 插件* 没有自动重置试用日期，因过期无法进入程序，此时无法再通过选择 “evaluate for free” 来重新试用，那么可以通过手动删除试用授权文件的方法来解决。

试用授权文件位于程序配置目录下的 *eval* 文件夹。

程序配置目录如下：

- windows：`%userprofile%/AppData/Roaming/JetBrains/产品名版本号`
- macos: `~/Library/'Application Support'/JetBrains/产品名版本号`
- linux: `~/.config/JetBrains/产品名版本号`

将程序配置路径下的 *eval* 文件夹 重命名或者删除后，重启程序即可。