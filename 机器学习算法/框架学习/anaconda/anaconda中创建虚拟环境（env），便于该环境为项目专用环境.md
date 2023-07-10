[参考博客地址](https://www.cnblogs.com/Sinte-Beuve/p/8597429.html)
- 最近在学习深度学习，将别人的代码下载到本地运行时，需要安装一大堆的依赖库，又担心各个库之间会产生冲突，于是需要在anaconda中新建虚拟环境，把这些需要的安装包都装到一个虚拟环境里面。这样，一个项目就对应一个虚拟环境，包与包之间也就不会产生冲突。
-  虚拟环境中常用操作命令
>conda env --help    #查看帮助
>conda env list  #列出所有的虚拟环境
>conda list --name [虚拟环境名]   #查看指定虚拟环境下的package
>#创建
>conda create --name [虚拟环境名] [python的版本] [需要的包]
>eg:
>conda create --name myenv
>conda create --name myenv python=2.7
>conda create --name myenv pytohon=2.7 numpy scipy
>#克隆
>conda create --name [虚拟环境名] -- clone [colne的环境]
>eg:
>#创建一个和原python环境一样的虚拟环境
>conda create --name mybase --clone base
>#删除
>conda remove --name [虚拟环境名] -all
>#激活取消（默认的环境是base）
>source activate [虚拟环境名]
>source deactivate [虚拟环境名]
>虚拟环境激活后，在cmd中输入python，显示的就是一个新的环境。
- 创建成功后，即可安装所需要的依赖包
> 可以通过  conda install [依赖包名]    或者   pip install [依赖包名] 进行安装。
- Note: 添加清华镜像源可以提高下载速度
>conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
>conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
>conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
>conda config --set show_channel_urls yes