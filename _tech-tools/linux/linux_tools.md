# Linux tools

## awk
awk 是一种处理文本文件的语言，是一个强大的文本分析工具。

[intro](https://www.runoob.com/linux/linux-comm-awk.html)

```
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```

examples:
```
计算文件大小
ls -l *.txt | awk '{sum+=$5} END {print sum}'

打印九九乘法表
seq 9 | sed 'H;g' | awk -v RS='' '{for(i=1;i<=NF;i++)printf("%dx%d=%d%s", i, NR, i*NR, i==NR?"\n":"\t")}'
```

## sed
sed 主要用来自动编辑一个或多个文件、简化对文件的反复操作、编写转换程序等。

[intro](https://www.runoob.com/linux/linux-comm-sed.html)

```
sed [-hnV][-e<script>][-f<script文件>][文本文件]
```

examples:
```
删除第2-5行
nl testfile | sed '2,5d'

打印ip地址
/sbin/ifconfig eth0 | grep 'inet addr' | sed 's/^.*addr://g' | sed 's/Bcast.*$//g'
```

## grep
grep 命令用于查找文件里符合条件的字符串。

[intro](https://www.runoob.com/linux/linux-comm-grep.html)

```
grep [-abcEFGhHilLnqrsvVwxy][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```

examples:
```
查找目录下包含"update"字符串的行
grep -r update /etc/acpi

反向查找，文件名包含test而内容不包含test的行
grep -v test *test*
```

## sort
sort 可针对文本文件的内容，以行为单位来排序。

[intro](https://www.runoob.com/linux/linux-comm-sort.html)

```
sort [-bcdfimMnr][-o<输出文件>][-t<分隔字符>][+<起始栏位>-<结束栏位>][--help][--verison][文件][-k field1[,field2]]
```

examples:
```
按第2列的值进行排序
sort testfile -k 2
```

## uniq
uniq 可检查文本文件中重复出现的行列。一般与 sort 命令结合使用。
当重复的行并不相邻时，uniq 命令是不起作用的。

[intro](https://www.runoob.com/linux/linux-comm-uniq.html)

```
uniq [-cdu][-f<栏位>][-s<字符位置>][-w<字符位置>][--help][--version][输入文件][输出文件]
```

examples:
```
统计各行在文件中出现的次数
sort testfile1 | uniq -c

在文件中找出重复的行
sort testfile1 | uniq -d
```

## xargs
xargs 能够捕获一个命令的输出，然后传递给另外一个命令。
xargs 一般是和管道一起使用。

[intro](https://www.runoob.com/linux/linux-comm-xargs.html)

```
somecommand |xargs -item  command
```

examples:
```
读取echo的字符串，重定义分隔符X，每行使用2个参数
echo "nameXnameXnameXname" | xargs -dX -n2

与find结合使用，可以一次删除很多文件，不用担心参数列表过长
find . -type f -name "*.log" -print0 | xargs -0 rm -f
```

## find
find 命令用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。如果使用该命令时，不设置任何参数，则 find 命令将在当前目录下查找子目录与文件。并且将查找到的子目录和文件全部进行显示。

[intro](https://www.runoob.com/linux/linux-comm-find.html)

```
find   path   -option   [   -print ]   [ -exec   -ok   command ]   {} \;
```

examples:
```
查找当前目录及子目录下后缀为".c"的文件
find . -name "*.c"

查找当前目录及子目录下所有文件
find . -type f

查找系统中所有文件长度为 0 的普通文件，并列出它们的完整路径
find / -type f -size 0 -exec ls -l {} \;
```

## wc
利用wc指令我们可以计算文件的Byte数、字数、或是列数，若不指定文件名称、或是所给予的文件名为"-"，则wc指令会从标准输入设备读取数据。

[intro](https://www.runoob.com/linux/linux-comm-wc.html)

```
wc [-clw][--help][--version][文件...]
```

examples:
```
统计三个文件的信息，3个数字分别表示文件的行数，单词数，字节数
wc testfile testfile_1 testfile_2
```
