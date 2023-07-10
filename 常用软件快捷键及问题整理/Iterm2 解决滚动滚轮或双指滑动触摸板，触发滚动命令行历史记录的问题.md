## Iterm2 解决滚动滚轮或双指滑动触摸板，触发滚动命令行历史记录的问题

* 在Iterm2中，如果上下滚动光标（上下滑动触摸板、或者滚动鼠标滚轮），通常情况下会触发屏幕内容上下滚动。
  但是在某些异常情况下，却触发了命令行历史记录的上下滚动，效果和你连续按了多次键盘的上下键按键一样。
* 如果想永久解决这问题，可在item2的“Preferences”->“advance”菜单中找到“wheel sends arrow keys when in alternate screen mode.”，并将该选项的“Yes”修改为“No”即可。