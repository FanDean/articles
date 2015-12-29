# Bash脚本

## Shebang

Shebang指文件第一行，用来指定命令解释器。

```bash
#! /bin/bash
```

解释器所在的位置也可以不写死，改用`/usr/bin/env`以$PATH确定执行哪一条命令。

```bash
#! /usr/bin/env node
```

脚本的退出值，0表示正常，1表示发生错误，2表示用法不对，126表示不是可执行脚本，127表示命令没有发现。如果脚本被信号N终止，则退出值为128 + N。

### set命令

`set`允许改变Shell的参数值，或设置新的参数。

```bash
# 打开bash的一个选项
set -LETTER

# 关闭bash的一个选项
set +LETTER

set -x # 打开debugging，执行的每条命令都会显示在终端
# your commands go here...
set +x # 关闭debugging

# 只要有一个命令或管道返回非零值，脚本就退出。
# 脚本的默认行为是忽略运行中的错误
# e 表示 errexit
set -e

# 只要脚本中有未定义的变量，脚本就会退出
set -u
```

下面的例子会输出`x`。

```javascript
foo() {
    echo 'x'
    return 42
}

out=$(foo)
echo $out
```

上面代码的顶部，加上`set -e`以后，就没有任何输出。

## Shell 扩展

大括号用于输出多个值。

```bash
$ echo last{a,b,c}.log
a.log b.log c.log
```

如果逗号前面没有值，就表示这是一个空字符串。

```bash
# 等同于
# cp a.log a.log.bak
$ cp a.log{,.bak}
```

大括号里面两个点号表示扩展。

```bash
$ echo {a..c}
a b c
```

多个大括号连用，会有循环处理的效果。

```bash
$ echo {a..c}{1..3}
a1 a2 a3 b1 b2 b3 c1 c2 c3
```

这个写法可以直接用于for循环。

```bash
for i in {1..4}
do
  echo $i
done
```

## 变量

### 概述

Bash脚本之中，可以自定义变量。变量使用等号赋值，等号前后都不能有空格。

```bash
$ a=3
```

对变量取值的时候，变量名前要加上美元符号。

```bash
$ echo $a
3
```

Bash变量名是大小写敏感的。变量名如果后面还紧跟着其他字符，可以将其放在大括号中。

```bash
INIT_DIR="${HOME}/.dotfiles/bash"
```

如果使用没有定义的变量，该变量的值为空字符。

```bash
$ cat v1.sh
#!/bin/sh
echo "Variable value is: $VAR1"
VAR1="Hello"
echo "Variable value is: $VAR1"

$ ./v1.sh
Variable value is:
Variable value is: Hello
```

Bash变量是弱类型的，可以随时改为其他类型的值。如果变量的值是字符串，而且包含空格，那么需要用双引号包起来。另外，双引号中的变量会被扩展成对应的值，单引号没有变量扩展的功能。

变量只对创建它的进程可见，除非使用`export`命令，将变量输出到子进程。执行脚本会新建一个子进程，因此脚本之中的变量不会泄漏到它的执行环境。

如果在Bash变量之前放上变量赋值语句，则该变量会输入子进程。

```bash
$ echo "$VAR5 / $VAR6"
 /
$ VAR5=5 VAR6="some value" bash
$ echo "$VAR5 / $VAR6"
5 / some value
$ exit
$ echo "$VAR5 / $VAR6"
 /
```

`declare`命令可以声明特定的命令。

```bash
$ declare option variablename
```

`declare`命令的参数为

- -r read only variable
- -i integer variable
- -a array variable
- -f for funtions
- -x declares and export to subsequent commands via the environment.

```bash
declare -i intvar
intvar=123
intvar=12.3 # 报错

declare -r rovar=281
rovar=212 # 报错
```

`readonly`命令可以设置只读变量。

```bash
$ readonly rov2="another constant value"
$ rov2=3
bash: rov2: readonly variable
```

`let`命令声明一个变量的值等于表达式的值。

```bash
$ let x=6-4
$ echo $x
2

$ x=6-4
echo $x
6-4
```

`local`关键字用于声明区块可见的局部代码。

```bash
pprint()
{
  local lvar="Local content"
  echo -e "Local variable value with in the function"
  echo $lvar
}
pprint
```

如果不加`local`，函数中声明的变量就是局部变量。

### 变量扩展

变量名前面加上`$`符号，就会输出变量的值。这就叫做“变量扩展”（parameter expansion）。

变量扩展有多种形式，可以修改并输出变量的值。

```bash
# 返回变量长度
${#string}

# 返回某个位置开始的子字符串
${string:position}

# 返回某个位置开始的、指定长度的子字符串
${string:position:length}

# 从字符串头部开始删除$substring的最短匹配
${string#substring}

# 从字符串尾部开始删除$substring的最短匹配
${string%substring}

# 从字符串头部开始删除最长匹配
${string##substring}

# 从字符串尾部开始删除最短匹配
${string%%substring}

# 字符串的查找和替换（仅替换第一个匹配）
${string/pattern/replacement}

# 替换所有匹配
${string//pattern/replacement}

# 替换字符串的头部匹配
${string/#pattern/replacement}

# 替换字符串的尾部匹配
${string/%pattern/replacement}

# 提取文件名的主体部分
echo ${filename#*.}

# 提取文件名的后缀名部分
echo ${filename%.*}
```

### 环境变量

- $PS1 提示符
- $SHELL 当前使用的Shell
- $$ 当前进程的ID
- $PPID 父进程的ID

```bash
$ echo $$
20708
# 新建一个子Shell
$ bash
$ echo $PPID
20708
# 退出子进程
$ exit
```

### 特殊变量

- $? 特殊变量，表示上一个命令的输出结果，如果是0，表示运行成功。
- $* 全部参数
- $0 脚本名
- $1 脚本的第一个参数
- $2 脚本的第二个参数，以此类推
- $# 参数数组的大小
- $$ 当前进程的ID
- $! 最近执行的背景线程的ID
- $- 使用`set`命令的参数
- $_ 上一个命令的最后一个参数。如果是shell脚本，就会给出绝对路径的文件名。

`$-`的例子。

```bash
set -e
# 输出 e
echo $-
```

参数的例子。

```bash
# hellokitty.sh
#!/usr/bin/env bash

echo hello $1

# 运行
$ ./hellokitty.sh kitty
hello kitty
```

## if结构

```bash
if condition
then
do something
else
do something else
fi
```

实例

```bash
$ a=joe
$ if [ $a == "joe" ]; then echo hello; fi
hello
```

脚本实例

```bash
#!/usr/bin/env bash

a=joe

if [ $a == "joe" ]; then
  echo hello;
elif [ $a == "doe" ]; then
  echo goodbye;
else
  echo "ni hao";
fi
```

进行判断的脚本。

```bash
$ mkdir testdir   # make a test directory
$ outputdir=testdir
$ if ! cd $outputdir; then echo "couldnt cd into output dir"; exit; fi
$ # no error -  now we're in the directory testdir
```

test是判断命令，也可以不把判断条件放在`[...]`之中。

```bash
if test $? -eq 0
then
fi

# 判断文件是否存在
if [ -e $file ]

# 判断文件大小是否为0
if [ -s $file ]
```

多个表达式的联合

- `[ ! EXPR ]` True if EXPR is false.
- `[ ( EXPR ) ]` Returns the value of EXPR. This may be used to override the normal precedence of operators.
- `[ EXPR1 -a EXPR2 ]` 表达式1和表达式2同时为`true`，则返回`true`
- `[ EXPR1 -o EXPR2 ]` 表达式1和表达式2之中只要有一个为`true`，则返回`true`
- `?(<PATTERN-LIST>)`	Matches zero or one occurrence of the given patterns
- `*(<PATTERN-LIST>)`	Matches zero or more occurrences of the given patterns
- `+(<PATTERN-LIST>)`	Matches one or more occurrences of the given patterns
- `@(<PATTERN-LIST>)`	Matches one of the given patterns
- `!(<PATTERN-LIST>)`	Matches anything except one of the given patterns

```bash
$ rm -f !(survivior.txt)
```

运算符

- -eq Is Equal To
- -ne Is Not Equal To
- -gt Is Greater Than
- -ge Is Greater Than or Equal To
- -lt Is Less Than
- -le Is Less Than or Equal To

字符串比较运算符

- == Is Equal To
- != Is Not Equal To
- < Is Less Than
- <= Is Less Than Or Equal To
- > Is Greater Than if
- >= Is Greater Than Or Equal To

常见的判断表达式。

- `[ -a FILE ]`	如果路径存在，返回`true`
- `[ -b FILE ]`	True if FILE exists and is a block-special file.
- `[ -c FILE ]`	True if FILE exists and is a character-special file.
- `[ -d FILE ]`	如果路径存在，而且是一个目录，返回`true`
- `[ -e FILE ]`	如果路径存在，返回`true`，否则返回`false`
- `[ -f FILE ]`	如果路径存在，而且是一个常规文件，返回`true`，否则返回`false`
- `[ -g FILE ]`	True if FILE exists and its SGID bit is set.
- `[ -h FILE ]`	True if FILE exists and is a symbolic link.
- `[ -k FILE ]`	True if FILE exists and its sticky bit is set.
- `[ -p FILE ]`	True if FILE exists and is a named pipe (FIFO).
- `[ -r FILE ]`	True if FILE exists and is readable.
- `[ -s FILE ]`	True if FILE exists and has a size greater than zero.
- `[ -t FD ]`	True if file descriptor FD is open and refers to a terminal.
- `[ -u FILE ]`	True if FILE exists and its SUID (set user ID) bit is set.
- `[ -w FILE ]`	True if FILE exists and is writable.
- `[ -x FILE ]`	True if FILE exists and is executable.
- `[ -O FILE ]`	True if FILE exists and is owned by the effective user ID.
- `[ -G FILE ]`	True if FILE exists and is owned by the effective group ID.
- `[ -L FILE ]`	True if FILE exists and is a symbolic link.
- `[ -N FILE ]`	True if FILE exists and has been modified since it was last read.
- `[ -S FILE ]`	True if FILE exists and is a socket.
- `[ FILE1 -nt FILE2 ]`	True if FILE1 has been changed more recently than FILE2, or if FILE1 exists and FILE2 does not.
- `[ FILE1 -ot FILE2 ]`	True if FILE1 is older than FILE2, or is FILE2 exists and FILE1 does not.
- `[ FILE1 -ef FILE2 ]`	True if FILE1 and FILE2 refer to the same device and inode numbers.
- `[ -o OPTIONNAME ]`	True if shell option "OPTIONNAME" is enabled.
- `[ -z STRING ]`	如果参数字符串的长度为0，则返回`true`，否则返回`false`
- `[ -n STRING ]` or `[ STRING ]`	True if the length of "STRING" is non-zero.
- `[ STRING1 == STRING2 ]`	True if the strings are equal. "=" may be used instead of "==" for strict POSIX compliance.
- `[ STRING1 != STRING2 ]`	True if the strings are not equal.
- `[ STRING1 < STRING2 ]`	True if "STRING1" sorts before "STRING2" lexicographically in the current locale.
- `[ STRING1 > STRING2 ]`	True if "STRING1" sorts after "STRING2" lexicographically in the current locale.
- `[ ARG1 OP ARG2 ]`	"OP" is one of -eq, -ne, -lt, -le, -gt or -ge. These arithmetic binary operators return true if "ARG1" is equal to, not equal to, less than, less than or equal to, greater than, or greater than or equal to "ARG2", respectively. "ARG1" and "ARG2" are integers.

## 循环

for循环

```bash
# 格式
$ for variable in list; do ... ; done

# 实例
$ for i in 1 2 3; do echo $i; done
1
2
3

$ for i in 1 2 hello; do echo $i; done
1
2
hello

$ for i in {1..10}; do echo -n "$i "; done; echo
1 2 3 4 5 6 7 8 9 10

$ for i in $( seq 1 2 10 ); do echo -n "$i "; done; echo
1 3 5 7 9
```

while循环

```bash
# 格式
$ while condition; do ... ; done

# 实例
$ x=1; while [ $x -lt 4 ]; do echo $x; x=$(($x+1)); done
1
2
3

$ cat junk.txt
1
2
3
$ while read x; do echo $x; done < junk.txt
1
2
3
```

## 正则运算符

`=~`是Bash的正则运算符，左侧运算数为等待验证的值，右侧为一个正则表达式。

```bash
if [[ $digit =~ [0-9] ]]; then
    echo "$digit is a digit"
else
    echo "oops"
fi
```

下面是另一个例子。

```bash
echo -n "Your answer> "
read REPLY
if [[ $REPLY =~ ^[0-9]+$ ]]; then
    echo Numeric
else
    echo Non-numeric
fi
```

```bash
files="*.jpg"
regex="[0-9]+_([a-z]+)_[0-9a-z]*"
for f in $files
do
    [[ $f =~ $regex ]]
    name="${BASH_REMATCH[1]}"
    echo "${name}.jpg"    # concatenate strings
    name="${name}.jpg"    # same thing stored in a variable
done
```

特殊字符

- `*`	Matches any string, including the null string (empty string)
- `?`	Matches any single character
- `X`	Matches the character X which can be any character that has no special meaning
- `\X`	Matches the character X, where the character's special meaning is stripped by the backslash
- `\\`	Matches a backslash

正则表达式的方括号。

- `[XYZ]`	The "normal" bracket expression, matching either X, Y or Z
- `[X-Z]`	A range expression: Matching all the characters from X to Y (your current locale, defines how the characters are sorted!)
- `[[:class:]]`	Matches all the characters defined by a POSIX® character class: alnum, alpha, ascii, blank, cntrl, digit, graph, lower, print, punct, space, upper, word and xdigit
- `[^…]`	A negating expression: It matches all the characters that are not in the bracket expression
- `[!…]`	Equivalent to `[^…]`
- `[]...]` or `[-…]` Used to include the characters ] and - into the set, they need to be the first characters after the opening bracket

## 执行脚本命令

在脚本中可以执行shell命令，有两种格式。

```bash
$ d=$(pwd)
$ d=`pwd`
```

实例。

```bash
$ len=$( cat file.txt | wc -l )
d=$( dirname $( readlink -m $0 ) )
```

## Process Substitution

Process Substitution（进程替换）也是用来在脚本中执行Shell命令，但是该Shell命令的输入和输出都是以文件形式出现。它的形式是`<( )`。

```bash
$ cat <( head -1 file.txt ) <( tail file.txt )
```

上面命令中cat的参数必须是文件，所以使用“进程替换”。

大括号也可以用来执行shell命令。

```bash
$ for i in 1 2 3; do { echo "***"$i; sleep 60 & } done
```

## 多行字符串

使用Here字符串，输出多行文本。

```bash
cat <<_EOF_

Usage:

$0 --inp [inputfile] --outputdir [dir]  --inst [number] --prefix [string] 

Required Arguments:

  --inp         the input file 

Options:

  --outputdir	the output directory (default: cwd)
  --inst	the number of instances requested (default: 1)
  --prefix	the prefix of output files (default: tmp)

Example:

$0 --inp tmp.txt --outputdir tmpdir --inst 10

_EOF_
```

## 文件处理

读取文件的命令格式。

```bash
while IFS= read -r line;
do
  COMMAND_on $line;
done < input.file
```

上面的`-r`参数表示防止反斜杠转义。

实例。

```bash
#!/bin/ksh
file="/home/vivek/data.txt"
while IFS= read -r line
do
  # display $line or do somthing with $line
  echo "$line"
done <"$file"
```

还可以读取每一行的栏位。

```bash
#!/bin/bash
file="/etc/passwd"
while IFS=: read -r f1 f2 f3 f4 f5 f6 f7
do
  # display fields using f1, f2,..,f7
  printf 'Username: %s, Shell: %s, Home Dir: %s\n' "$f1" "$f7" "$f6"
done <"$file"
```

## 函数

函数定义方式如下。

```bash
function functionname()
{
commands
.
.
}
```

命令行调用函数方式如下。

```bash
$ functionname arg1 arg2
```

Bash函数的退出状态，就是函数体内最后一个命令的退出状态。

Bash提供一些特殊变量，用于读取函数变量。

- $# 参数个数
- $@ 所有参数，等同于`$0 $1 ...`
- $0 脚本名
- $1、$2、$3 函数各个参数

```bash
$ testfunc () { echo "$# parameters"; echo "$@"; }
$ testfunc
0 parameters

$ testfunc a b c
3 parameters
a b c
$ testfunc a "b c"
2 parameters
a b c
```

根据函数用户相似的，就是为命令起别名。

```bash
$ alias name='unix command with options'
```

## 实例

### 查看指定目录大小

```bash
#!/bin/bash
#Author Leo G

echo " Enter your directory: "
read x
du -sh "$x"
```

### 备份脚本

```bash
#!/bin/bash
#Author Leo G

rsync -avz  -e "ssh " /path/to/yourfile user@backupserver.com:/backup/

echo "backup for $(date) "| mail -s "backup complete" user@youremail.com
```