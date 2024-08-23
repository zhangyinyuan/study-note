# Linux常用技巧

### 每次vim的时候总是粘贴不了.需要输入:set mouse-=a回车,才行
解决办法, vim ~/.vimrc 内容如下
```
set mouse-=a
```
保存后, ```souce ~/.vimrc```, 再次编辑文件, 就可以粘贴了

