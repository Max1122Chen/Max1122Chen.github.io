# The Missing Semester

## 1. Shell

命令行工具

Shell中的语言是可编程的，可以实现脚本

当在命令行中调用程序时，通过“环境变量”中的“Path”变量指明的路径信息寻找对应的程序

通过which [程序名] 可以找出对应程序的路径

Linux和MacOS都只有一个根目录，Windows每个磁盘目录有一个根目录（C:\）

~ 会展开成根目录

cd - 可以切换到之前所在的一个目录

程序的命令参数可以这样区分：

-XXX 后不带参数是flag

-XXX后带参数是option

-l 用于输出详细信息，在MacOS或Linux系统下，ls -l可以展示文件和目录的权限

read权限决定了用户是否能列出文件或目录

write权限决定了用户是否能写入、删除文件或目录

execute权限决定了用户是否能“打开”或“进入”一个目录或文件，不止是“执行”

CTRL + L 清屏

### 管道符

"|"把其左侧程序的输出作为右侧程序的输入

### Root User

拥有最高权限

获取最高权限：sudo——Super User do

sudo可以让你跑到sys文件夹中，对系统做一些神秘的事情

$符号意味着当前不是root用户

#符号意味着是root

sudo su可以把当前用户切换成root

### xdg-open

xdg-open会用合适的软件打开参数中的文件



### Programing in Shell

#### variable

```bash
foo=bar
echo $foo

output: bar
```

使用echo输出纯字符串时，使用双引号或单引号包围是一样的，但如果输出的内容中包含$var，使用双引号会展开var的内容，单引号则不会

```bash
foo=bar
echo "Hello $foo"

output: Hello bar

echo 'Hello $foo'
output: Hello $foo
```

#### function

source [filename] 可以把.sh文件中的东西加载到Shell中，这允许我们在文件中定义函数来封装多条Shell命令，并且还可以使用参数



$_可以展开成上一次使用的参数

$?会展开成一个表示error的标志，0代表正常，1代表error



#### Logic Ops

可以用逻辑运算符来连接命令，此时的true和false实际上的含义是”是否执行成功“



#### $( )

可以用$( )套住一个命令（程序），它会把程序的输出存起来



#### *

*是一种通配符，常用用法有用于查找某种后缀的文件

  

#### {}

{}中包围的内容可以和前面的内容拼起来，展开成多段参数

e.g.

```bash
echo foo{1,2}
# is same to
echo foo1 foo2
```



## 2. Vim

vim是一个基于”模态“的编辑器，这是说它有多种模式



## 3. Data wrangling



## 4. Command Line Environment



## 5. Version Control System

Git使用有向无环图存储历史记录，而非线性结构，允许从一个父记录分支出多个子记录，以并行修改



Git把文件称作blob，目录称作tree

commit也是Git中一个重要的数据结构概念，其中包含了其”继承“的commit，作者名字，message，tree

blob, tree, commit都被视作object

object会拥有哈希值作为标识用于访问



git log查看提交日志

git checkout ( ) 转移到指定的版本快照/指定的分支，实际上在改变工作目录

git diff [ ]查看当前版本和其他版本存在的区别，可以指定版本的哈希值。并且不止可以检查当前版本和其他版本的区别，还可以指定两个任意版本进行比较。比较时当然可以指定比较具体的文件而不只是整个快照之间比较



### Branching

branch允许开发者基于master创建其他并行开发分支

git branch 打印所有分支

git branch [newBranchName] 创建一个新分支

git checkout [branchName] 转移到一个分支



### Merging

merge将master之外的分支合并回master，有时可以直接合并，有时需要解决合并冲突。

git可以将冲突的地方标记出来，开发者只需手动解决它。实际上还有更高级的工具用于解决冲突



### Remote

git remote 可以列出所有和当前仓库有关的远程仓库，这不止是关联GitHub这样的远程托管平台，还可以关联到本地电脑上的其他git仓库

git remote add <name> <url> 添加远程仓库，name常用的有"origin"

git push <name> <branchName>

git branch --set-upstream-to=<remoteRepoName/branchName> 可以设置推送的默认目标远程仓库

git fetch <remoteRepoName> 可以把远程仓库的内容拉取到本地，但不意味着会直接覆盖本地的内容，本地用户可以选择用merge合并远程仓库发生的修改

git pull 则是结合fetch和merge的命令



## 6. Debug & Profiling

### Log



## 7. Meta Programing

元编程是对编程的“描述性“的内容

Build System用于构建一些东西，其描述了”如何构建“



