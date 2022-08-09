## Python

### 1.0 argparse模块

argparse 是 Python 内置的一个用于命令项选项与参数解析的模块，通过在程序中定义好我们需要的参数，argparse 将会从 sys.argv 中解析出这些参数，并自动生成帮助和使用信息。

```
import argparse

# 1、创建 ArgumentParser() 对象
parser = argparse.ArgumentParser()

# 2、调用 add_argument() 方法添加参数
parser.add_argument('integer',type = int,help = 'display an integer')
parser.add_argument('str',type = ascii,help = 'display an string')

# 3、使用 parse_args() 解析添加的参数
args = parser.parse_args()

print(args.integer)
print(args.str)
```

add_argument() 方法

```
ArgumentParser.add_argument(name or flags...[, action][, nargs][, const][, default][, type][, choices][, required][, help][, metavar][, dest])
```

```
每个参数解释如下:

name or flags - 选项字符串的名字或者列表，例如 foo 或者 -f, --foo。
action - 命令行遇到参数时的动作，默认值是 store。
store_const，表示赋值为const；
append，将遇到的值存储成列表，也就是如果参数重复则会保存多个值;
append_const，将参数规范中定义的一个值保存到一个列表；
count，存储遇到的次数；此外，也可以继承 argparse.Action 自定义参数解析；
nargs - 应该读取的命令行参数个数，可以是具体的数字，或者是?号，当不指定值时对于 Positional argument 使用 default，对于 Optional argument 使用 const；或者是 * 号，表示 0 或多个参数；或者是 + 号表示 1 或多个参数。
const - action 和 nargs 所需要的常量值。
default - 不指定参数时的默认值。
type - 命令行参数应该被转换成的类型。
choices - 参数可允许的值的一个容器。
required - 可选参数是否可以省略 (仅针对可选参数)。
help - 参数的帮助信息，当指定为 argparse.SUPPRESS 时表示不显示该参数的帮助信息.
metavar - 在 usage 说明中的参数名称，对于必选参数默认就是参数名称，对于可选参数默认是全大写的参数名称.
dest - 解析后的参数名称，默认情况下，对于可选参数选取最长的名称，中划线转换为下划线.
```

### 2.0 sys模块

#### 2.0.1 sys.argv

给程序在外部传递参数

**脚本的名称总是sys.argv列表的第一个参数**

```
源码：
import sys

print sys.argv[0]
print sys.argv[1]
print sys.argv[2]
print sys.argv[3]

执行命令：
python test.py arg1 arg2 arg3

输出：
test.py
arg1
arg2
arg3
```

#### 2.0.2 sys.platform

根据不同平台适配不同调用方法

```
os_type = sys.platform
print(os_type)
if os_type == 'win32':
    os.system('dir') #调用windows的命令窗口。
elif os_type == 'mac':
    os.system('ls') #调用mac的命令窗口。
elif os_type == 'linux':
    os.system('shell') #调用linux
```

#### 2.0.3 sys.exit

需要中途退出程序时调用

```
sys.exit(0) #正常退出
```

#### 2.0.4 sys.path

包含输入模块的目录名列表

```
print(sys.path)
```

#### 2.0.5 sys.modules

This is a dictionary that maps module names to modules which have already been loaded. This can be manipulated to force reloading of modules and other tricks.

```
for names in sys.modules.keys():
	print(names)
```

#### 2.0.6 sys.stdin,sys.stdout,sys.stderr

**stdin**，**stdout**，以及 **stderr** 变量包含与标准I/O 流对应的流对象。如果需要更好地控制输出，而 print 不能满足你的要求，它们就是你所需要的。 你也可以替换它们，这时候你就可以重定向输出和输入到其它设备( device )，或者以非标准的方式处理它们。

### 3.0 subprocess 模块

**run 方法语法格式如下：**

```
subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None, universal_newlines=None)
```

- args：表示要执行的命令。必须是一个字符串，字符串参数列表。
- stdin、stdout 和 stderr：子进程的标准输入、输出和错误。其值可以是 subprocess.PIPE、subprocess.DEVNULL、一个已经存在的文件描述符、已经打开的文件对象或者 None。subprocess.PIPE 表示为子进程创建新的管道。subprocess.DEVNULL 表示使用 os.devnull。默认使用的是 None，表示什么都不做。另外，stderr 可以合并到 stdout 里一起输出。
- timeout：设置命令超时时间。如果命令执行时间超时，子进程将被杀死，并弹出 TimeoutExpired 异常。
- check：如果该参数设置为 True，并且进程退出状态码不是 0，则弹 出 CalledProcessError 异常。
- encoding: 如果指定了该参数，则 stdin、stdout 和 stderr 可以接收字符串数据，并以该编码方式编码。否则只接收 bytes 类型的数据。
- shell：如果该参数为 True，将通过操作系统的 shell 执行指定的命令。

```
#执行ls -l /dev/null 命令
>>> subprocess.run(["ls", "-l", "/dev/null"])
crw-rw-rw-  1 root  wheel    3,   2  5  4 13:34 /dev/null
CompletedProcess(args=['ls', '-l', '/dev/null'], returncode=0)
```

returncode: 执行完子进程状态，通常返回状态为0则表明它已经运行完毕，若值为负值 "-N",表明子进程被终。

```
import subprocess
def runcmd(command):
    ret = subprocess.run(command,shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE,encoding="utf-8",timeout=1)
    if ret.returncode == 0:
        print("success:",ret)
    else:
        print("error:",ret)


runcmd(["dir","/b"])#序列参数
runcmd("exit 1")#字符串参数
```

### 4.0 pathlib.Path 模块

```
from pathlib2 import Path

#当前所在路径
p = Path.cwd()
print("current path:%s" % p)

type(p.glob(r"*"))
print("Path模块下的 glob(*)：")
# 返回：当前目录中的所有文件和文件夹
for i in p.glob(r"*"):
    print(i)
    
print(f"\nPath模块下的 glob(**)：")
# 返回：当前目录，及其下所有子目录中的所有文件夹
for i in p.glob(r"**"):
    print(i)
 
print("-"*80)
 
print(f"\nPath模块下的 rglob(*)：")
# 返回：当前目录，及所有子目录中的所有文件和文件夹
for i in p.rglob("*"):
    print(i)
    
print(f"\nPath模块下的 rglob(**)：")
# 返回：当前目录，及其下所有子目录中的所有文件夹
# 相当于 glob(**)
for i in p.rglob("**"):
    print(i) 
```

### 5.0 read file

read()： 一次性读取整个文件内容。推荐使用read(size)方法，size越大运行时间越长

readline()：每次读取一行内容。内存不够时使用，一般不太用

readlines() ：一次性读取整个文件内容，并按行返回到list，方便我们遍历


### 6.0 add copyright information

```
from pathlib import Path

# file headers contains copyright information
notification = "/* BCST - Introduction to Computer Systems\n" \
" * Author:      lyrics@outlook.com\n" \
" * Github:      https://github.com/lyrics/bcst_csapp\n" \
" * Bilibili:    https://space.bilibili.com/4564101\n" \
" * Zhihu:       https://www.zhihu.com/people/zhao-yang-min\n" \
" * This project (code repository and videos) is exclusively owned by lyrics \n" \
" * and shall not be used for commercial and profitting purpose \n" \
" * without lyrics's permission.\n" \
" */\n\n"

def add_copyright_header():
    # get files with paths
    filelist = list(Path(".").rglob("*.[ch]"))

    # recursively add lines to every .c and .h file
    print("recursively add lines to every .c and .h file")
    for filename in filelist:
        try:
            with open(filename, "r", encoding = 'ascii') as fr:
                content = fr.read()
                if (content.startswith("/* BCST - Introduction to Computer Systems")):
                    print("\tskip\t%s" % filename)
                    fr.close()
                    continue
                else:
                    fr.close()
                    # reopen and write data: this is a safer approach
                    # try to not open in r+ mode
                    print("\tprepend\t%s" % filename)
                    with open(filename, "w", encoding = 'ascii') as fw:
                        fw.write(notification + content)
                        fw.close()
        except UnicodeDecodeError:
            print(filename)
			
add_copyright_header()
```
