
GUN Make -> Cmake

# GUN Make(Makefile)
```ad-Reading
[概述 — 跟我一起写Makefile 1.0 文档](https://seisman.github.io/how-to-write-makefile/overview.html)
```

>无论是C还是C++，首先要把源文件（.cpp/.c）编译成中间代码文件(.obj/.o),即Object File。然后再把大量Object File合成可执行文件，这个动作叫链接

>举个简单的例子，再liunx下面，如果我有三个文件，main.cpp,source.cpp,source.h.其中，再编译阶段，应该把main.cpp和source.cpp全部编译成Object File，然后在一同链接在一起生成可执行文件。

```bash
g++ -std=c++17  main.cpp source.cpp -o main
```


>源文件首先会生成中间目标文件，再由中间目标文件生成可执行文件。在编译时，编译器只检测程序语法和函数、变量是否被声明。如果函数未被声明，编译器会给出一个警告，但可以生成Object File。而在链接程序时，链接器会在所有的Object File中找寻函数的实现，如果找不到，那到就会报链接错误码（Linker Error）

所有，如果源文件很多个使用g++命令直接编译就会超级麻烦所以我们引入了Makefile

```makefile
target ...: prerequistites ...
[Tab]precipe
	...
	...
```

^66e7a0

**target** :Object file or Excute file of label
**prerequistites** : 生成该target所依赖的文件或target
**recipe**:该target需要执行的命令(shell)
**makefile用#做注释**
```ad-important
title:Makefile的核心规则
prerequisites中如果至少一个文件笔target文件要新的画，recipe所定义的命令就会被执行。
```

课上的keyword count 案例
```ad-example
```makefile
edit:main.o count.o
	g++ -std=c++ main.o count.o -o edit

main.o:main.cpp count.h
	g++ -std=c++ -c main.cpp

count.o:count.cpp count.h
	g++ -std=c++ -c count.cpp

clear:
	rm main.o count.o
```

```ad-important
title:makefile的工作逻辑
按照依赖关系，递归完成任务。
比如上述这个例子
首先寻址`edit`的依赖文件`main.o` 和 `count.o`;
如果发现就执行`g++ ...`,否则就去生成`*.o`文件;
首先根据`main.o`的依赖关系，因为我们已经有了`main.cpp`和`count.h`[如果没完成这两个文件，先把代码写了再来鸭！！]，通过`g++ ...`生成`main.o`;
然后根据`count.o`的依赖关系，生成`count.o`
好了，现在我们有了`edit`所需要的文件，从而`g++ ...`生成文件！

提一嘴：`clear`的冒号后面没有东西，表明该target不依赖于任何文件，从而我们可以用`make clear`显式的调用这个target对应的shell命令。
```

```ad-tip
title:Makefile中使用变量
objects = main.o count.h \\可以简化文本内容
edit : $(objetc) \\使用 $(变量名)使用变量
```

````ad-important
title:让make自动推导
GUN的make功能强大，可以自动推导文件已经文件依赖关系后面的命令。
从而上面的代码可以改成
```makefile
objects = main.o count.o
edit:$(objects)
	g++ $(objects) -o edit

main.o:main.cpp count.h
count.o:count.cpp count.h


.PHONY:clear
clear:
	rm main.o count.o
````

````ad-note
title:Makefile 的另一种风格
但这种风格的文件依赖关系不那么明显，比如这里所有的objects都依赖于count.h，让后main.o还依赖于main.cpp,count.o还依赖于count.cpp
```
objects = main.o count.o
edit:$(objects)
	g++ $(objects) -o edit

$(objects):count.h
main.o : main.cpp
count.o : count.cpp

.PHONY:clear
clear:
	-rm main.o count.o
````

````ad-note
title:清空目录的规则
一个makefile最后一定要有一个`clear`命令 
现在我们来看看我们一直忽略的几行代码
```Makefile
.PHONY:clear
clear:
	-rm edit $(objects)
````

^ec4172

## Part I 书写规则
[[#^66e7a0|基本模板]]

**通配符** ,make支持三种通配符`*` `?` `~` [规则和bash一致]
`*.c`表示为所有后缀为.c的文件

````ad-example
title:通配符
```makefile
clear:
	cat main.c
	rm -f *.o
````

````ad-attention
注意写法 `:=`以及`wildcard`是make的关键字之一
```makefile
objects = *.o #表明object的值(value)就是*.o，而不是会展开文件名。
objects := $(wildcard *.o) #此时object的值就是所有.o文件，会展开。
````

````ad-note
title:文件搜寻
make默认会从当前目录搜索依赖文件，可以通过`VPATH`【特殊变量】指明make的搜寻其他目录
或者使用关键字`vpath` 
```makefile
VPATH = src:../headers #会再本目录搜素不到的时候去src下的headers搜寻

vpath %.h ../headers #从../headers目录下搜素所有.h文件 
#vpath必须包含%符号
````

````ad-note
title:vapth的执行顺序
如果存在重复了的<pattern>会有按照语句在makefile的顺序进行搜索
```makefile
vpath %.c foo
vpath %.c blish
vpath %.c bar #会按照foo->blish->bar搜索
```
````

```ad-note
title:伪目标
![[#^ec4172|伪目标的一个例子]]
其中`.PHONY`显式的指明这个目标为`伪目标`,伪目标不是一个文件而是一个标签`label`,make无法生成它的依赖关系和决定它是否要执行。我们只有显式地指明这个目标才能使其生效。
```

```makefile
all : prog1 prog2 prg3
.PHONY : all #只用一个make指令生成一堆可执行文件

prog1 : ...
....
```

```makefile
#静态模式
<targets ...> : <target-pattern> : <prereq-patterns ...>
	<commands>
	....

#example
objects = foo.o bar.o

all: $(objects)

$(objects): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
```

^89af4d

`%.o` 表面从`objects`获取所有以`.o`为结尾的文件， `%.c`分两步1.`%`取出上述获取的`.o`文件的文件名也就是取出`foo bar` 2.把所有文件名加上`.c`
其中`$<` `$@`是自动化变量，含义为`表面第一个依赖文件` `目标集`

```bash
#自动生成依赖关系
#GUN的C/C++编译器 `-MM`参数就会输出某文件依赖文件，包含标准库,如果要显示标准库需要使用-M
g++ -std=c++17 -MM main.cpp

#output:xxx.h ....
```

![[Pasted image 20240430181718.png|实际效果]]

````ad-note
title:自动生成依赖关系
通过上面可知，使用GUN可以生成某文件的依赖关系，哪如何把这些消息直接加入makefile文件呢。答案是：没办法(...)就挺无语的吧，但可以曲线救国，比如可以为每一个name.cpp文件生成一个name.d的makefile文件，`.d`文件存放对应`.c`文件的依赖关系。参考代码如下
```makefile
%.d: %.cpp
    @set -e; rm -f $@; \
    $(g++) -M $(CPPFLAGS) $< > $@.$$$$; \
    sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
    rm -f $@.$$$$
````

上面代码的含义是：所有的 `.d` 文件依赖于 `.c` 文件， `rm -f $@` 的意思是删除所有的目标，也就是 `.d` 文件，第二行的意思是，为每个依赖文件 `$<` ，也就是 `.c` 文件生成依赖文件， `$@` 表示模式 `%.d` 文件，如果有一个C文件是name.c，那么 `%` 就是 `name` ， `$$$$` 意为一个随机编号，第二行生成的文件有可能是“name.d.12345”，第三行使用sed命令做了一个替换，关于sed命令的用法请参看相关的使用文档。第四行就是删除临时文件。
```makefile
#总的来说是把
main.o :main.c defs.h
#转换为
main.o main.d : main.c defs.h
```


## Part II 书写命令
`@...` 默认情况下面，make一个makefile一个文件的时候会输出正在执行的指令，但我们可以使用`@`指令隐藏输出.`each 正在编译xxxx模块`会输出`each 正在编译xxxx模块` 但如果`@each 正在编译xxxx模块` 则只会输出`正在编译xxxx模块`

在make的时候`-n` `-just-print` 只显示命令，而不执行命令。

````ad-attention
title:命令执行
如果某条指令的执行依赖于上一条指令执行的结果，则不能把这条指令写在上一条指令的另一行，而应该用`;`并且把指令写在后面，如下
```makefile
exec:
	cd /~/hchen
	pwd #并不会输出 ~/hchen #而是会输出当前目录
exec:
	cd /~/hchen; pwd #会正确输出
````

````ad-note
title:命令出错：忽略 or 退出
通常情况下，make会逐行执行并检测每条指令的返回值，然后一旦出现错误就会终止执行本次make，但有的时候我们并不希望出现错误就立刻结束，比如说`mkdir`我们只是为了确保存在某个文件夹，但如果该文件夹以及存在make会报错，但我们并不需要终止，所以跳过这个`错误`很重要。所以引入了`-`，标记为`-`的命令即是出错也不影响其他命令的执行。
```makefile
clear:
	-rm -f *.o #表明即是rm命令失败，也不会影响其他命令，注意后面-f是参数的格式而不是忽略错误。
````

make的时候加上`-i`或`--ignore-errors`参数，本次make忽略所有错误，而如果某个规则以`IGNORE`为目标，则该规则内的所有错误都将被忽略。
make还可以加上`-k`或者`--keep-going`某规则出错终止该规则，但其他规则依旧运行。

[嵌套执行make](https://seisman.github.io/how-to-write-makefile/recipes.html#:~:text=%E6%89%A7%E8%A1%8C%E5%85%B6%E5%AE%83%E8%A7%84%E5%88%99%E3%80%82-,%E5%B5%8C%E5%A5%97%E6%89%A7%E8%A1%8Cmake,-%C2%B6) [暂时用不到，多文件复杂工程才需要]

````ad-note
title:定义命令包
用`define xxxx` `endef`所包容的多条命令
```makefile
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef

foo.c : foo.y
	$(run-yacc)
````

^296e2e

`define`后面跟的就是命令包的名字，不能和已经定义的变量重名。使用命令包的方法`$(name)`和使用变量一致。


## Part III 变量
>makefile 中的变量类似于C/C++中的宏一样，代表的是一个文本字串。
>不同于宏，可以修改变量的值，并且允许以数字开头，同时包含字符、数字、字母、下划线。但不允许包含`:`、`#`、`=`、或者空字符。命名规范：驼峰命名法。

>变量在声明的时候必须被赋初值，使用的时候用`$`并用`()`或者`{}`把变量括起来。

````ad-note
title:变量的定义
两种办法`=`和`:=`.
前者：可以使用暂时未被赋值的变量定义某变量，也就是可以用后面的变量定义变量。
后者：前面的变量不能使用后面的变量，只能使用前面已经定义了的变量。
```makefile
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all:
    echo $(foo)
    #会打印 Huh? 
````

````ad-attention
title:`#`的一个特性
如下，这个变量的值是`/foo/bar`+四个空格[变量定义终止于#]
```makefile
dir := /foo/bar| | | | |#
````
`?=` 等价于
```makefile
FOO ?= bar#等价于
fieq ($(origin FOO), undefined)
	FOO = bar
endif
```

`````ad-note
title:变量的高级用法
````ad-note
title:变量值的替换
`$(var:a=b)`或者`${var:a=b}`,把变量`var`中所有以`a`结尾的字符串中的`a`替换为`b`,结尾的意思是空格或者结束符。
![[#^89af4d|格式]]
```makefile
foo := a.o b.o c.o
bar := $(foo:.o=.c) #把foo中所有以.o结尾的文件名替换为.c
#使用静态模式
bar := $(foo:%.o=%.c)#模式必须包含%，作用和上面一致。
`````

`把变量值再当成变量`
```makefile
x = y # = 可以使用后面的变量
y = z
a := $($(x)) #此时a的值是z

#一个有趣的案例
first_second = hello
a = first
b = second
all = $($a_$b) #$故(all) == hello 可以把变量拼接起来
```

`追加变量值 +=`
```makefile
objects = main.o
objects += another.o

$(objects) == main.o another.o 
#如果该变量未定义则+=被转换为= ，并且+=会保留原变量声明的符号`=`或者`:=`
```

`多行变量` 类似于![[#^296e2e]]
不过，变量不能用`Tab`开头，否则make认为这是一条命令。
```makefile
define tow-lines
echo foo    #变量名
echo $(bar) #变量值
endef

等价于
echo = echo
foo = $(bar) #注意只能是`=`
```


`局部变量`(Target-specific Variable)之前的所有变量都是`全局变量`
`局部变量`允许与全局变量名称一致，其值只会再某条规则内有效。
```makefile
<target ...> : <variable-assignment> #允许=,:=,?=,+=
<target ...> : overide <variable-assignment>#用于修改make命令行带入的变量，或者是系统环境的变量。
```

```makefile
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
    $(CC) $(CFLAGS) prog.o foo.o bar.o

prog.o : prog.c
    $(CC) $(CFLAGS) prog.c

foo.o : foo.c
    $(CC) $(CFLAGS) foo.c

bar.o : bar.c
    $(CC) $(CFLAGS) bar.c #不管全局变量GFLAGS是什么，这条规则内所有的GFLAGS都是-g
```

`模式变量`
```makefile
%.o : GFLAGS = -O
```
