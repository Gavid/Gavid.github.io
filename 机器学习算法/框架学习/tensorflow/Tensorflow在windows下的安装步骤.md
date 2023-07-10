@[TOC](Tensorflow在windows下的安装步骤)
# 1. 首先在window系统中下载、安装anaconda
- anaconda下载地址(https://www.anaconda.com/distribution/)
- anaconda安装教程与使用教程(https://zhuanlan.zhihu.com/p/32925500)、（https://blog.csdn.net/ITLearnHall/article/details/81708148）
# 2. 打开[Anaconda Prompt]、输入< ==anaconda search -t conda tensorflow==  > 对tensorflow版本以及适用系统进行搜索，
- ![Anaconda Prompt](https://img-blog.csdnimg.cn/20190703141638440.png)
- ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703142042260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
# 3. 找到适用于 windows 的tensorflow 的版本Name(左边方框内).
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703142624398.png)
# 4. 输入< ==anaconda show *nwani/tensorflow-eigen(版本Name)*== >,搜索该版本的安装命令。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703143013732.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
# 5. 复制安装命令，回车，安装中。。。。
# 6. 打开jupyter notebook,测试安装是否成功。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190703143202869.png)
# Note : 若在Linux或macos系统下，也可以直接使用< ==pip install tensorflow== > 进行安装。
