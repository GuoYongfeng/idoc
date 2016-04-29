<h1 style="font-size: 40px;text-align:center;color: #007cdc;">
    基本SHELL命令使用
</h1>

## 1.ls 显示文件

命令格式：`ls [option] file`

```
# 显示详细列表
$ ls -l
$ 显示所有文件，包含隐藏文件（以. 起头的文件名）
$ ls -a
# 显示文件及所有子目录
$ ls -R
$ 显示文件（后跟*）和目录（后跟/）
$ ls -F
# 与l选项合用，显示目录名而非其内容
$ ls -d
```

## 2.cd 目录切换

注意目录分隔符为“/”，与dos相反

```
$ cd dirname
# 到d盘的website目录
$ cd /d/website
$ cd ../
$ cd ../../
```

## 3.pwd 显示当前路径

```
$ pwd
```
## 4.cat 显示文件内容

```
$ cat filename
```

## 5.rm 删除文件或目录

命令格式： `rm [-r] filename (filename 可为档名，或档名缩写符号.)`
```
# 删除档名为 file1 之文档.
$ rm file1
# 删除档名中有五个字元，前四个字元为file 之所有文档.
$ rm file?
# 删除档名中，以 f 为字首之所有文档.
$ rm f*
# 删除目录 dir1，及其下所有文档及子目录.
$ rm -r dir1
```

## 6.mkdir 新建文件夹

```
$ mkdir dirname
$ mkdir website
```

## 7.cp 文档的复制

命令格式: `cp [-r] source destination`

```
# 将文档 file1 复制成 file2
$ cp file1 file2
# 将文档 file1 复制到目录 dir1 下，文件名仍为 file1.
$ cp file1 dir1
# 将目录 /tmp 下的文档 file1复制到现行目录下，文件名仍为 file1.
$ cp /tmp/file1 .
# 将目录 /tmp 下的文档 file1现行目录下，档名
为file2
$ cp /tmp/file1 file2
# (recursive copy) 复制整个目录.
$ cp -r dir1 dir2
# 复制dir1整个目录到dir2
$ cp -R dir1 dir2
```

## 8.mv 移动文件

命令格式： mv source destination
```
# 将文档 file1，更改档名为 file2.
$ mv file1 file2
# 将文档 file1，移到目录 dir1 下，档名仍为 file1.
$ mv file1 dir1
# 若目录 dir2 不存在，则将目录 dir1，及其所有档案和子目录，移到目录 dir2 下，新目录名称为 dir1.若目录dir2 不存在，则将dir1，及其所有文档和子目录，更改为目录 dir2.
$ mv dir1 dir2
```

## 9.grep 搜索

```
$ grep string filename
# 在a.js文件中查找===字符串，并且显示行号
$ grep -rn '====' a.js
```

## 10.vim 编辑器

命令状态：

- j,k,h,l:上下左右
- 0： 行首
- $: 行尾
- i,I :插入命令，i 在当前光标处插入 I 行首插入
- a,A:追加命令，a 在当前光标后追加，A 在行末追加
- o,O:打开命令，o 在当前行下打开一行，O在当前行上插入一行
- r,R :替换命令，r 替换当前光标处字符，R从光标处开始替换
- 数字s: 替换指定数量字符
- x: 删除光标处字符
- dd: 删除当前行
- d0: 删除光标前半行
- d$: 删除光标后半行
- ctrl+f :后翻页
- ctrl+b:前翻页
- G : 文件尾
- 数字G: 数字所指定行
- /string 查找字符串
- n 继续查找
- N 反向继续查找
- % 查找对应括号
- u 取消上次操作
- ex命令状态
- ：0 文件首
- ：1,5 copy 7 块拷贝
- ：1，5 del 块删除
- ：1，5 move 7 块移动
- ：1，$s/string1/string2/g 全文件查找string1并替换为string2
- ：wq! 存盘退出

## 结语

以上内容属于常用的基础shell命令，不完整，但简单实用。系统深入学习请继续查找相关资料。