1. #### bash的概念

   一个bash就是一个线程,在bash里执行bash会打开一个子线程

2. #### 文本流及重定向

   0: 标准输入流

   1: 标准输出流

   2: 错误输出流

   `>`重定向

   `>>`追加

   `ls / /aabb 1>ls.out 2>ls.out`同时有1和2的时候,会先输出错误信息,最终结果是记录1的内容,因为2错误会先输入,然后1输出会覆盖2的输出

   `ls / /aabb 2>&1 1>ls.out`会先在控制台打印错误信息,之后把输出流写到文件里

   `ls / /aabb 1>ls.out 2>&1`会把错误信息和输出信息先后写入到文件里,`1>ls.out 2>&1`特殊写法有`&> ls.out`或者`>& ls.out`

   `>`符号左边不能有空格.

   **输入重定向:**

   `read aaa<<<"hello sxt"`

   `read aaa<<AABB`

   `cat 0</etc/initab`

3. #### shell脚本编程

   - 访问百度的例子:

   `cd /prod/$$/fd`查看当前线程的文件描述符(一般含有0,1,2,225)

   `exec 8<> /dev/tcp/www.baidu.com/80`开通一个socket链接到百度

   `echo -e "GET / HTTP/1.0\n" 1>&8`其中`-e`作用激活转义字符,把请求输出到8这个文件描述符(发送到百度服务器)

   `cat 0<&8`cat把输入打印到控制台,把8文件描述符的数据输入到控制台

   `exit`只能通过退出当前bash,并重新连接来关闭8这个文件描述符(或者说只是重开一个bash)

   - 变量

   `LANG=zh_CN.UTF-8`设置当前bash变量`LANG`的值为`zh_CN.UTF-8`(设成中文)

   ```shell
   # sxt(){			#定义一个函数
   > echo sxt			#打印值sxt	
   > echo $abc			#打印名字为abc的变量
   > local bbb=hello	#设置方法内变量bbb的值为hello--本地变量
   > echo $bbb			#打印变量bbb的值
   > }
   # abc=xxx			#将xxx赋值给变量abc--局部变量
   # sxt				#调用sxt()方法
   sxt					#打印sxt值
   xxx					#打印$abc变量的值xxx	
   hello				#打印变量bbb的值hello
   ```

   位置变量`$1,$2,${11}`,运行`a.sh`

   ```shell
   # cat a.sh
   echo $1
   echo $2
   # source a.sh a b
   a
   b
   ```

   数组变量

   ```shell
   # sxt=(1,2,3)		# 这种写法是错误的
   # sxt=(1 2 3)		# 正确写法
   # echo ${sxt}		# 取出数组
   1 2 3
   # echo ${sxt[1]}	# 取出指定位置变量
   2
   ```

   特殊变量:

   `$#`位置参数的个数

   `$?`上个命令退出的转态,0成功,非0失败

   `$*`或者`$@`参数占位符,例`${sxt[*]}`,`${sxt[@]}`会显示数组中所有的元素

   `$$`当前shell的PID:接收者(和`$BASHPID`一样)

   在管道里创建子bash可以直接使用父bash的变量

   可执行文件不能使用父bash的变量,只能使用全局变量,可以通过`export 变量名`使局部变量变成全局变量

   `$$`和`$BASHPID`的区别:

   在管道中使用`$$`会调用父bash的id,`$BASHPID`能显示真正的管道bash id.

   ```shell
   流程控制
   if
   while
   for
   seq 字符序列
   vi 中使用:set nu 可显示行号
   ```

   