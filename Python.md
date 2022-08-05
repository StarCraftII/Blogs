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
