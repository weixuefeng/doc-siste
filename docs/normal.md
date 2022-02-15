# mac 指定文件夹搜索

## 1. 打开终端

![image-20220113115544707](/Users/weixuefeng/Library/Application Support/typora-user-images/image-20220113115544707.png)

## 2. 查看当前位置

``` 
$ pwd
```

## 3. 进入指定文件夹

```
$ cd backup/
```



![image-20220113115724342](/Users/weixuefeng/Library/Application Support/typora-user-images/image-20220113115724342.png)

如果是外接移动硬盘的目录，一般在 `Volumes` 下面，可以使用 `ls` 命令查看到你的硬盘信息，直接  `cd` 进去就可以

![image-20220113115826973](/Users/weixuefeng/Library/Application Support/typora-user-images/image-20220113115826973.png)

## 4. 查找文件，模糊搜索

```
$ find . -name "*名称*"
```

比如搜索 `我不是吴奇隆的音频`，可以这样搜索

```
$ find . -name "*吴奇隆*"
```

