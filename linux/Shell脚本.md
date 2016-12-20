# Shell脚本

---

部分内容暂未记录

---

### 数组和关联数组

数组，借助索引将多个独立的数据存储为一个集合。关联数组，采用字符串式索引的数组。

定义数组：`array_var={1 2 3 4 5 6}`

或将数组定义为一组“索引-值”

```shell
array_var[0]="var1"
array_var[1]="var2"
array_var[2]="var3"
array_var[3]="var4"
```

打印特定索引的数组元素内容

```shell
index=3
echo ${array_var[$index]}
```

打印数组中的所有值、数组长度

`echo ${array_var[*]}`	`echo ${#array_var[*]}`

定义关联数组

首先将一个变量声明为关联数组`declare -A ass_array`

声明之后将元素添加到关联数组中

```
ass_atrray={[index1]=val1 [index2]=val2}
或
ass_array[index1]=val1
ass_array[index2]=val2
```

显示数组内容`ecsho ${array_var[index]}`

列出数组索引`echo ${!array_var[*]}`或`echo ${!array_var[@]}`

### 使用别名

别名是一种便捷方式，省去输入一长串命令序列。

`alias new _command ='command sequence'`

`alias install ='sudo apt-get install'`

`unalias`删除别名

例：`alias rm='cp $@ ~/backup && rm $@'`此时rm为备份并删除文件的别名

对别名进行转义

使用`\command`对别名 进行转义，从而忽略当前定义的别名

### 获取终端信息

tput 和stty 是用于终端处理的工具

获取终端的行数和列数

```
tput cols
tput lines
```

打印当前终端名`tput longname`

将光标移动到坐标(100,100)处`tput cap 100 100`

设置终端颜色`tputsetb n`	n可以为0~7之间的值

设置文本前景色`tputserf n`

设置文本样式为粗体`tput bold`

设置下划线的起止`tput smul` 	`tput rmul`

`stty -echo `	禁止将输出发送到终端	 `stty -echo echo`允许将输出发送到终端

### 获取、设置日期和延时

使用不同的格式来读取、设置日期

读取日期`data`	打印纪元时`data +%s`

日期转换为纪元时`data --data "Thu Nov 18 08:00:00 IST 2008" +%s `

星期数 `data --data "Jan 20 2009 +%a`

控制输出格式 `data "+%d %B %Y` 	格式 为 日 月 年

设置日期和时间 data -s "格式化的日期字符串"

`data -s "21 June 2008 11:11:11"`

检查一组命令所花费的时间

```
start=$(data +%s）
...
end=$(data +%s)
difference=$((end -start))
```

或者使用`time<scriptpath>`来得到执行脚本所花费的时间

延时:`sleep`推迟命令的执行

### 调试脚本

运行脚本时使用`-x`选项，启用shell脚本的跟踪调试功能

`bash -x script.sh`

使用`set -x` 和`set +x`对脚本进行部分调试

传递_DEBUG环境变量自定义的调试信息

```
#!/bin/bash
funtion DEBUG(){
  [ "$_DEBUG" == "ON" ] && $@ ||;
}
for i in {1..10}
do
	DEBUG echo $i
done
```

运行时将调试功能置为"on"来运行上面的脚本

_DEBUG=on ./script.

`set -v`	当命令进入读取模式时显示

`set +v` 禁止打印输入

打shebang从`#!/bin/bash`改成`#!/bin/bash -xv`	

### 函数和参数

定义函数

```
function fname(){statements}
#或
fname(){statements}
```

调用函数	`fname`

传递参数	`fname arg2 arg2`

使用参数：`$0` 脚本名

`$1`  第一个参数  `$n` 第n个参数   `$@`   被扩展成`"$1","$2","$3" t等` 

`$*`被扩扩展成`$1c $2c $3c`等，其中c是IFS中的第一个字符，所有的参数都当做单个字符。

递归函数

​	Fork炸弹`:(){ :|:& };:`

**注解如下：**

**:() **# 定义函数,函数名为":",即每当输入":"时就会自动调用{}内代码

**{ **# ":"函数开始标识

**: **# 用递归方式调用":"函数本身

**| **# 并用管道(pipe)将其输出引至...

**: **# 另一次递归调用的":"函数

\# 综上,":|:"表示的即是每次调用函数":"的时候就会生成两份拷贝

**& **# 调用间脱钩,以使最初的":"函数被杀死后为其所调用的两个":"函数还能继续执行

**} **# ":"函数结束标识

**; **# ":"函数定义结束后将要进行的操作...

**: **# 调用":"函数,"引爆"fork炸弹

导出函数		`export`

函数也能像环境变量一样被`export` 导出，函数的作用域可以扩展到子进程中

读取命令返回值

```
cmd;
echo $?;
```

`$?`会给出命令`cmd`的返回值，返回值被称为退出状态，如果命令成功退出，则退出状态为0，否则为非0

```
#!/bin/bash
CMD="command" #command指代需要检测退出状态的目标命令
$CMD
if[ $? -eq 0];
then
	echo "successfully"
else
	echo "unsuccessfully"
fi
```

向命令传递参数

```
command -p -v -k 1 file
command -pv -k 1 file
command -file -pvk 1
command -vpk 1 file
```

### 将命令序列的输出读入变量

输入通常是通过`stdin`或参数传递给命令，输出则出现在`stderr`或`stdout`，组合多个命令时通常将`stdin`用于输入，`stdout`用于输出

此时，这些命令被称为过滤器，使用管道连接每个过滤器，管道操作符是`|`

例：`cmd1 | cmd2 |cmd3`

cmd1的输出传递给cmd2，cmd2的输出传递给cmd3，最终的输出（来自cmd3）将会被打印或导入某个文件。

`ls | cat -n out.txt`		ls的输出被传递给`cat -n` 后者通过`stdin`所接收到的输入内容加上行号，然后将输出重定向到文件`out.txt`

读取由管道相连的命令序列的输出：`cmd_output=$(COMMANDS)`,这种方法被称为**子shell**，另一种方式被称为**反引用**

```
cmd_output=`COMMANDS`
echo cmd_output
```

给命令分组的方法：

1、利用子shell生成一个独立的进程，可以使用`{}`操作符来定义一个子shell

```
pwd;
{cd /bin; ls};
pwd;
#当命令在子shell中执行时，不会对当前shell有任何影响
```

2、通过引用子shell的方式保留空格和换行符

```
test.txt
1
2
3

out=$(cat text.txt)
#out 为 1	 2		3
out="$(cat text.txt)"
#out 为 1
		2
		3

```

### 不使用回车键来读取n个字符

从输入中读取n个字符并存入变量variable_name						`read -n  number_of_chars variable_name`

用无回显的方式读取密码`read -s var`

显示提示信息`read -p "Enter input :" var`

在特定时限内读取输入`read -t timeout var`

用特定的定界符作为输入行的结束`read -d "var" var`

### 运行命令直至执行成功

```
repeat()
{
	while true
	#true可以用":" 代替，因为true是作为/bin中一个二进制文件来实现的，这意味着每执行一次while循环都需要生成一个进程，":"是shell内建的命令，总是会返回为0的代码
	do
		$@ && return
		#在循环中执行以参数形式传入的命令，
		#如果执行成功则返回，进而退出循环
		sleep 30;
		#使命令延时30秒
	done
}
```

### 字段分隔符和迭代器

**内部字段分隔符（IFS）**是用于特定用途的定界符。**IFS**J是存储定界符的环境变量。是当前shell环境使用的默认定界字符串。

我们需要迭代一个字符串或**逗号分隔型数值CSV**中的单词，如果是前一种，则使用IFS="."，如果是后一种，则使用IFS=“,”。

使用IFS读取变量中的第一个条目

```
data="name,sex,rollno,location"
oldIFS=$IFS
IFS=, NOW,
do echo item : $item
done
IFS= $oldIFS
```

IFS的默认值为空白字符（换行符、制表符或空格），当IFS被置为逗号时，shell将逗号视为一个定界符。

循环结构：

```
for var in list 
do 
	echo $ var
done
```

其中`list`可以是一个字符串，也可以是一个序列。`echo {1..50}`能够生成一个从1~50的数字列表，`echo {a..z}`或`echo {A..Z}`或`echo {a..h}`可以生成字母列表。

```
for (int i =0;i<count;i++){
  commands
}
```

```
while condition	#用true作为循环条件则产生无限循环
do
	commands
done
```

```
until condition
do
	commands
done
```

### 比较与测试

```
if condition
then
	commadns
else if condition then
		commands
	else
		commands
fi
```

使用运算符简化判断，例

```
[ condition ] && action #逻辑与，如果condition为真，则执行action
[ condition ] || action #逻辑或，如果condition为假，则执行action
```

算术比较

条件通常被放置在封闭的中括号内。在`[` 或`]`与操作数之间有一个空格。

````
[ $var -eq 0 ]	#var的值等于0时返回真
[ $var -ne 0 ]  #var的值为非0时返回真
-gt	#大于
-lt	#小于
-ge	#大于或等于
-le	#小于或等于
#多个条件结合
[ $var -ne 0 -a $var -gt 2 ]	#-a 逻辑与
[ $var -ne 0 -o $var -gt 2 ]	#-o 逻辑或
````

文件系统相关测试

```
-f $file_var	给定变量包含正常的文件路径或文件名则返回真
-x $var		给定变量包含的文件可执行则返回真
-d $var		给定变量包含的是目录则返回真
-e $var		给定变量包含的文件存在则返回真
-c $var		给定变量包含的是一个字符设备文件的路径则返回真
-b $var		给定变量包含的是一个块设备文件的路径则返回真
-w $var		给定变量包含的文件可写则返回真
-r $var		给定变量包含的文件可读则返回真
-L $var		给定变量包含的是一个符号链接则返回真
```

---

## 命令

### 用`cat`进行拼接

`cat`通常用于读取、显示或拼接文件内容。

`cat`打印文件内容		`cat file1.txt file2.txt`

`cat`使**管道操作符**从标准输入中读取	`OUTPUT_FROM_SOME COMMANDS | cat`

`echo "Text through stdin" |cat -file.txt` 

`-`被作为stdin文本的文件名？

`cat -s ` 		删除所有连续的空白行

`cat -T`		将制表符标记成`^I`

`cat -nb` 	在每一行内容之前加上行号，`b`选项跳过空白行

### 录制并回放终端会话

`script -t 2> timing.log -a output.session`

其中：timing.log用于存储时序信息，描述每一个命令在何时运行，output.session用于存储命令输出。`-t`选项用于将时序数据导入stderr，`2>`则用于将stderr重定向到timming.log。

回放命令执行过程

`scriptreplay timing.log putput.session`

### 文件查找与文件列表

`find`的应用场景和基本用法

`find base_path`  从 base_path开始向下查找

`find . -print` 	打印出匹配文件的文件名（路径），'\n'作为用于对输出的文件名进行分隔		-print0指明使用‘\0’作为匹配的文件名之间的定界符，用于文件名中含有换行符的状况

`find PATH -name **.txt -print`		根据文件名或正则表达式进行搜索，选项`-name`的参数指定了文件名所必须匹配的字符串，支持通配符。`iname`选项为匹配文件名但是忽略大小写

匹配多个条件中的一个，可以采用`OR`条件操作

`find . \( -name "*.txt" -0 -name "*.pdf" \) -print`

`\(   \)`用于将多个条件作为一个整体

`-path`选项的参数可以使用通配符来匹配文件路径。`-name`总是用给定的文件名进行匹配，`-path`则将文件路径作为一个整体进行匹配。`-regex`与`-path`相似，不过`-regex`的参数是基于正则表达式来匹配文件路径的

用`!`来否定参数的含义		`find . ! -name "*.txt" -print`

基于目录的深度搜索	采用深度选项`-maxdepth`和`-mindepth`来限制find命令遍历的目录深度。只允许find在当前目录中查找，深度可以设置为1，需要向下两级时，深度可以设置为2

`find . -maxdepth 1 -name "f*" -print`

`-maxdepth`和`-mindepth`应该作为find的第三个参数出现，如果作为第四或之后的参数，可能会影响到find的效率

根据文件类型搜索

Unix类操作系统将一切都视为文件，文件具有不同的类型，例如普通文件、目录、字符设备、块设备、符号链接、硬链接、套接字以及FIFO等。

`-type`可以对文件搜索进行过滤，为find命令指明特定的文件匹配类型。

```
find . -type d #当前文件夹下所有目录
find . -type f #当前文件夹下所有普通文件
find . -type l #符号链接
find . -type s #套接字
find . -type p #FIFO
```

根据文件时间进行搜索

Unix/Linux文件系统中每一个文件都有三种时间戳

**访问时间**-atime 用户最后一次访问文件的时间

**修改时间-mtime** 文件内容最后一次被修改的时间

**变化时间-ctime** 文件元数据(权限或所有权)最后一次改变的时间

```
find . -type f -atime -7	#最近7天内被访问过的文件
find . -type f -atime 7		#七天前被访问过的文件
find . -type f -atime +7	#最近访问时间超过七天的文件
```

`-never`参数：指定一个用于比较时间戳的参考文件，然后找出比参考文件更新所有文件

`find . -type f -never file.txt`		列出比file.txt修改时间更近的所有文件

基于文件大小的搜索

```
find . -type f -size +2k	#列出当前目录下大小大于2k的文件
find . -type f -size -2k	#列出当前目录下大小小于2k的文件
find . -type f size 2k		#列出当前目录下大小为2k的文件
#其它文件大小单元
#b--块（512字节）	c--字节	w--	字（2字节）k--（1024字节）	
#M--1024k	G--1024M
```

删除匹配的文件

`-delete`可以用来删除`find`找到的所有文件

`find . -type f -name "*.swap" -delete` 

基于文件权限和所有权的匹配

`find . -type f - perm 644 -print`		打印出权限为644的文件

根据文件所有权进行搜索

`find . type f -user username `

利用`find`执行命令或动作

`find`命令可以借助选项`-exec`与其它命令进行结合

`find . -type f -user root -exec chown username {} \;`

在这个命令中`{}`是一个与`-exec ` 选项搭配使用的特殊字符串，对于每一个匹配的文件，`{}`会被替换成相应的文件名。

`find . -type f -name "*.c" -exec cat {} \;> cfiles.txt`

`-exec`无法直接使用多个命令，为了执行多个命令可以把命令写到shell脚本中，然后执行这个脚本`-exec ./commands.sh {}\;`

`-exec`结合`printf`来生成有用的输出信息

`find . -type f -nam "*.txt" -exec printf "Text file : %s \n"  { } \ `

find跳过指定的目录

`find PATH \{ -name ".git" -prune \} -o \{ -type f -print\}`

## xargs

将标准输入数据重新格式化，再将其作为参数提供给其他命令。xargs能够处理stdin并将其转换为特定命令行参数，也可以将单行或多行文本输入 转换成其它格式，例如单行变多行或多行变单行。

`args`命令应该紧跟在管道操作符之后，以标准输入转作为主要的源数据流，它使用`stdin`并通过提供命令行参数来执行其它命令。

`cat example.txt | xargs` 将多行输入转换成单行输出 

`cat example.txt | xargs -n 3`	将单行划分成多行

`xargs`可以用自己的定界符来分隔参数，用`-d`选项为输入来指定一个定制的定界符		`echo splitxsplitxsplitxsplit | xargs -d x`

使用`-d`选项将"x"作为输入定界符（默认情况下使用内部字段分隔符IFS作为输入定

**结合find使用xargs**

`find . -type f -name "*.txt" -print0 | xargs -0 rm -f` 

删除所有的.txt文件	`xargs -0`将`\0` 作为输入定界符

**统计源代码目录中所有C程序文件的行数**

`find sourcecode_dir  -type f -name "*.c" - print0 | xargs -0 wc -l `

`wc -l`显示一个文件的行数

**结合stdin，巧妙运用while语句和子shell**

```
cat files.txt | (while read args;do cat $arg; done )
#等同于cat files.txt | xargs -I {} cat {}
```

在while循环中，可以将`cat $args `替换成任意数量的命令，从而可以对同一个参数执行多条命令，也可以不借助管道将输出传递给其它命令。

`cmd0 | (cmd1;cmd2;cmd3) | cmd4`

### 用tr进行转换

`tr` 可以对来自标准输入的内容进行字符替换、字符删除以及重复字符压缩。

tr只能通过stdin(标准输入)而无法通过命令行来接受输入

`tr [options] set1 set2`

将来自stdin的输入字符从set1映射到set2，然后将输出写入stdout（标准输出）set1和set2是字符类或字符集，如果两个字符集的长度不相等，那么set2会不断重复最后一个字符，直到长度与set1相同，如果set2的长度大于set1,那么在set2中超出set1长度的部分字符会被全部忽略。

tr可以完成**大小写转换，文本替换，删除字符，字符集补集，压缩字符等功能**

### 校验和与核实

校验和(checksum)程序用于从文件中生成校验和密钥，然后利用这个校验和密钥核实文件的完整性。

### 加密工具与散列

`crypt`  `gpg` ` base64` `md5sum` `sha1sum` `openssl`

### 排序、唯一与重复

`sort`命令能够帮助我们对文件文件和stdin进行排序操作。`uniq`的作用是从文本或stdin中提取出唯一（或）重复的行。

sort`  命令既可以从特定文件，也可以从stdin中获取输入，并将输出写入stdout.`uniq`的工作方式和`sort`一样。

```
sort file1.txt file2.txt > sorted.txt
sort file1.txt file2.txt -o sorted.txt
```

按照数字顺序进行排序

`sort -n file.txt`

按照逆序进行排序

`sort -r file.txt`

按照月份进行排序

`sort -M nouths.txt`

合并两个已排序过的文件

`sort -m sorted1 sorted2`

找出已排序文件中不重复的行

`sorte file1.txt file2.txt | uniq`

检查文件是否已经排序过

`sort -c filename`      如果文件已经排序，`sort`会返回为0的退出码，否则返回非0，`$?`中保存上次运行命令的返回码

依据键或列进行排序

`sort-nrk 1 data.txt	#对data.txt按第一列按逆序`

指定某个范围内的字符为键

`sort -nk 2,3 data.txt`

为了使sort的输出与以`\0`作为终止符的xargs命令相兼容

`sort -z data.txt | xargs -0`

`uniq`命令通过消除重复内容，从给定输入中（stdin或命令行参数）找出唯一的行,`uniq`只能用于排序过的数据，一般与`sort`结合使用

`uniq sorted.txt`

`sort unsorted.txt | uniq -u`

统计各行在文件中出现的次数

`sort unsorted.txt | uniq -c`

找出文件中重复的行

`sort unsorted.txt | uniq -d`

`-s`  指定前面忽略的字符数 `-w`指定用于比较的最大字符数

`sort data.txt | -s 2 -w 2`  

### 临时文件命名与随机数

存储临时数据，存放在`/tmp`目录中，重启后会被清空

创建临时文件	

```
filename=`mktemp`
echo $filename
```

创建临时目录

```
dirname=`mktemp -d`
echo $dirname
```

生成文件名，但不创建文件、目录

```
tmpfile=`mktemp -u`
echo $tmpfile
#文件名存储在tempfile中，但并没有创建对应的文件
```

根据模板创建临时文件名

`mktemp test.XXX`

### 分割文件和数据

**文件大小和文件内容分割**

`split -b 10k data.txt` 分割文件,以`xaa xab xabc` 形式命名，`-b` 的参数是分割文件的大小其单位可以为`M(MB)/G(GB)/c(byte)/w(word)`

`-d`选项指定以数字为后缀  `-a length` 指定后缀长度

指定文件名前缀为`splite_file` 

`spli -b 10k data.file -d -a 4 split_file`

`csplit`  依据指定条件和字符串匹配选项对文件进行分割

`/str/`用来匹配某一行，分割的过程即从此处开始

`/[REGEX]/`表示文本样式，包括从当前行直到（但不包括）

包含`/str/`的匹配行

`{*}`表示根据匹配重复进行分割，直到文件末，可以用`{整数}`来指定分割执行的次数

`-s`使命令进入静默模式，不打印其它信息

`-n`指定分割后的文件名后缀的数字个数，例如01，02等

`-f`指定分割后的文件名的前缀

`-b`指定后缀格式，文件名= 前缀+后缀

**根据扩展名切分文件名**

借助`%`操作符可以轻松将名称部分从“名称.扩展名”这种格式中提取出来

```
file_jpg="sample.jpg"
name=${file_jpg%.*}		#name为sample,删除了 . 后面的内容
extension=${file_jpg#*.} #extension的jpg，删除了 . 前面的内容
```

`file=/dir1/dir2/dir3/my.file.txt`

`${file#*/}`：删掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt
`${file##*/}`：删掉最后一个 /  及其左边的字符串：my.file.txt
`${file#*.}`：删掉第一个 .  及其左边的字符串：file.txt
`${file##*.}`：删掉最后一个 .  及其左边的字符串：txt
`${file%/*}`：删掉最后一个  /  及其右边的字符串：/dir1/dir2/dir3
`${file%%/*}`：删掉第一个 /  及其右边的字符串：(空值)
`${file%.*}`：删掉最后一个  .  及其右边的字符串：/dir1/dir2/dir3/my.file
`${file%%.*}`：删掉第一个  .   及其右边的字符串：/dir1/dir2/dir3/my

`${file:5:5}`：提取左侧第五个字符起的五个字符

`${file/dir/path}`：将第一个dir 替换为path：/path1/dir2/dir3/my.file.txt
`${file//dir/path}`：将全部dir 替换为 path：/path1/path2/path3/my.file.txt

**工作原理**

`${VAR%.*}`的含义

从`$VAR`中删除位于%右侧通配符所匹配的字符串，通配符从右向左进行匹配

%属于非贪婪模式，它从右到左找出匹配通配符的最短结果，`%%`与`%`相似，但行为模式是贪婪的，匹配符合条件的最长的字符串

`${VAR#*.}`

从`$VAR`中删除

---

### 批量重命名和移动

用特定的格式重命名当前目录下的图像文件

```
#!/bin/bash
count=1;
for img in `find . -inmae '*.png'-o -iname '*.jpg' 
-type f -maxdepth 1`
do
	new=image-${count}.${img##*.}
	echo rename $img to $new
	mv "$img" "$new"
	let count++
done
```

