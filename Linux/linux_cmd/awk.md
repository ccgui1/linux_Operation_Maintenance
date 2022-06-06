# awk命令  
## 一、概述  
awk是专门为文本处理设计的编程语言，其中文件的每行数据称之为**记录**, 默认以空格或制表符为**分隔符**，每条记录被分成若干**字段（列）**，awk每次从文件种读取一条记录。  

## 二、基础语法
### 2.1 语法格式
```shell
awk [选项] '条件{动作}  条件{动作} ... ...'  文件名
```

### 2.2 pirnt指令  
可以输入常量和变量，如果时字符串常量需要用双引号括起来，数字常量可以直接打印。
- -F   
***默认是以空格、换行符、制表符为分隔符***，使用-F可以指定分隔符。
#### -F $n使用
```bash
[root@localhost ~]# awk  -F : '{ print "username: " $1 }'   /etc/passwd
username: root
username: bin
username: daemon
username: adm
username: lp
username: sync
username: shutdown
username: halt
```



### 2.3  内置变量
awk语法由一系列条件和动作组成，在**花括号**内可以有多个动作，多个动作之间用分号分隔，在多个条件和动作之间可以有若干空格，也可以没有。

如果没有指定条件则匹配所有数据，如果没有指定动作则默认为print打印。   

$$常见awk内置变量$$     

| 变量名  | 描述  |
| :---    | :----|
| FIELNAME | 当前输入文档的名称 |
| FNR    |  当前输入文档的当前行号，尤其当有多个输入文档时有用|
| NR  | 当前输入数据流的当前行号 |
| $0  | 当前行的全部数据内容  | 
| $n...  | 当前行的第n个字段的内容（n>=1）|
| NF    | 当前记录（行）的字段（列）个数  |
| FS    | 字段分隔符，默认为空格或者Tab制表符  |
| OFS   | 输出字段分隔符，默认空格  |
| ORS   | 输出记录分隔符，默认为换行符\n  |
| RS    | 输入记录分隔符，默认为换行符\n | 



#### FILENAME  
```bash
# 列出当前文件的名字
[root@localhost ~]# awk  -F : '{ print "filername: " FILENAME }'   /etc/passwd
filename: /etc/passwd
filename: /etc/passwd
filename: /etc/passwd
filename: /etc/passwd
filename: /etc/passwd
```


#### FNR和 NR
本质上来讲就是输出每个文件处理的行号,FNR是按文件来分，新读取一个文件时，行号就从头开始，而NR可以理解成，先将多个文件整合成一个大文件，然后输出的是这一个大文件的内容的行号
```bash
# FNR
[root@localhost ~]# awk  -F : '{ print "filename: " FILENAME "  当前文件的行号  "  FNR}'   /etc/passwd   /etc/hosts
filename: /etc/passwd  当前文件的行号  1
filename: /etc/passwd  当前文件的行号  2
filename: /etc/passwd  当前文件的行号  3
filename: /etc/passwd  当前文件的行号  4
.....
filename: /etc/passwd  当前文件的行号  47
filename: /etc/passwd  当前文件的行号  48
filename: /etc/hosts  当前文件的行号  1
filename: /etc/hosts  当前文件的行号  2

# NR
[root@localhost ~]# awk  -F : '{ print "filename: " FILENAME "  当前文件的行号  "  NR}'   /etc/passwd   /etc/hosts
filename: /etc/passwd  当前文件的行号  1
filename: /etc/passwd  当前文件的行号  2
filename: /etc/passwd  当前文件的行号  3
filename: /etc/passwd  当前文件的行号  4
filename: /etc/passwd  当前文件的行号  5
.....
filename: /etc/passwd  当前文件的行号  45
filename: /etc/passwd  当前文件的行号  46
filename: /etc/passwd  当前文件的行号  47
filename: /etc/passwd  当前文件的行号  48
filename: /etc/hosts  当前文件的行号  49
filename: /etc/hosts  当前文件的行号  50

```
#### \$0和\$n   
n代表的是一个数值，代表获取第一列的值
```bash
# $n
[root@localhost ~]# awk  -F : '{ print "用户名: " $1}'   /etc/passwd 
用户名: root
用户名: bin
用户名: daemon
用户名: adm
用户名: lp
用户名: sync
用户名: shutdown
```
```bash
# $0
[root@localhost ~]# awk  -F : '{ print $0}'   /etc/passwd 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```   

#### NF   
输出当前行有几个字段 **（注意利用这个特性，我们可以取文件的最后一列的值）**

```bash
# 每行按: 分隔都是7个字段
[root@localhost ~]# awk  -F : '{ print NF}'   /etc/passwd 
7
7
7
7
7
```
 
```bash
# 利用$n的特性，取出每一行的最后一列。
[root@localhost ~]# awk  -F : '{ print $NF}'   /etc/passwd 
/bin/bash
/sbin/nologin
/sbin/nologin
/sbin/nologin
/sbin/nologin
/bin/sync
/sbin/shutdown
/sbin/halt
/sbin/nologin
/sbin/nologin
/sbin/nologin
```

#### FS   
打印当前字段使用的分隔符，默认是空格或Tab制表符。
```bash
[root@localhost ~]# awk  -F : '{ print FS}'   /etc/passwd 
:
:
:
:
```


#### RS,ORS,FS,OFS
##### RS和ORS
RS保存的是输入数据的行分隔符，默认\n，可以指定其他字符作为行分隔符。 ORS保存的是输出字段的行分隔符
```bash
# RS
[root@localhost ~]# awk   '{ print RS}'   /etc/passwd 
 


 

#ORS
[root@localhost ~]# awk  -v ORS="《----》"   '{print  $1}'   /etc/passwd
root:x:0:0:root:/root:/bin/bash《----》bin:x:1:1:bin:/bin:/sbin/nologin《----》daemon:x:2:2:daemon:/sbin:/sbin/nologin《----》adm:x:3:4:adm:/var/adm:/sbin/nologin《----》lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin《----》
```




```bash
# 这里使用-v重新定义了RS的值，那么就导致了默认的行分隔符,由\n,变成了.
# -v 变量名=变量值，用来定义变量值。
[root@localhost ~]# awk  -v RS='.'   '{print  $1}'   /etc/hosts 
127
0
0
1
localdomain
localdomain4
localdomain
localdomain6

```
##### OFS和FS
OFS保存的是输出字段的分隔符（列分隔符），默认空格。FS是输入字符的列分隔符（效果和-F 一样）
```bash
# 此处使用-v重新定义了FS的分隔符为:(其实就是定义了按:来划分字段)
[root@localhost ~]# awk  -v FS=':' -v OFS="_"   '{print  $1 OFS $2}'   /etc/passwd
root_x
bin_x
daemon_x
adm_x
lp_x
sync_x
shutdown_x
halt_x
mail_x
operator_x
```


### 2.4 条件匹配  
常见的匹配符号

| 比较符号| 描述  |
|:------ | :------|
|/匹配值/ | 全行数据正则匹配 | 
|!/匹配值/ |对全行数据正则匹配后取反 |
| ~/匹配值/ | 对特定数据正则匹配   |
| !~/匹配值/  | 对特定数据正则匹配后取反  |
| ==  |等于  |
| !=   | 不等于   |
| >    | 大于    |
| >=   | 大于等于   | 
| <     | 小于     |
| <=    | 小于等于  |
| &&    |  逻辑与   |
| ||    | 逻辑或   |   



```bash
# 从所有行中过滤出含有指定字段的行
[root@localhost ~]# awk    '/bash/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
student:x:1000:1000:student:/home/student:/bin/bash



# 从指定列中过滤出含有指定字段的行
# 分别从第一列和最后一列进行过滤。
[root@localhost ~]# awk  -v FS=':'   '$1~/a/' /etc/passwd
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
halt:x:7:0:halt:/sbin:/sbin/halt
......
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
[root@localhost ~]# awk  -v FS=':'   '$NF~/a/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
halt:x:7:0:halt:/sbin:/sbin/halt
student:x:1000:1000:student:/home/student:/bin/bash



# 过滤出第三列的值大于999的行
# 也就是uid>999的用户信息。
[root@localhost ~]# awk  -v FS=':'   '$3>999' /etc/passwd
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
student:x:1000:1000:student:/home/student:/bin/bash

```

### 2.5 BEGIN和END
BEGIN导致动作指令仅在读取任何数据记录之前执行一次，END导致动作指令仅在读取所有数据记录后执行一次   
简单来说：BEGIN可以进行数据初始化，END可以进行数据汇总。

```bash
# 其中
[root@localhost ~]# awk -v FS=":" 'BEGIN{print "用户名 uid  解释器"} {printf("%-20s %-10s %-10s\n",$1,$3,$NF)} END{print "共有"NR"个账号"}' /etc/passwd
用户名 uid  解释器
root                 0          /bin/bash 
bin                  1          /sbin/nologin
daemon               2          /sbin/nologin
adm                  3          /sbin/nologin
lp                   4          /sbin/nologin
sync                 5          /bin/sync 
....
tcpdump              72         /sbin/nologin
student              1000       /bin/bash 
apache               48         /sbin/nologin
libvirtdbus          975        /sbin/nologin
共有48个账号

```

### 2.6 数字计算

```bash
# 加分
[root@localhost ~]# awk "{print 2+3}"  /etc/hosts 
5
5
[root@localhost ~]# awk "BEGIN{print 2+3}" 
5
# 减法
[root@localhost ~]# awk "BEGIN{print 2-3}" 
-1
 
# 乘法
[root@localhost ~]# awk "BEGIN{print 2*3}" 
6

# 除法
[root@localhost ~]# awk "BEGIN{print 2/3}" 
0.666667

# 取余数
[root@localhost ~]# awk "BEGIN{print 2%3}" 
2
[root@localhost ~]# awk "BEGIN{print 5%3}" 
2
[root@localhost ~]# awk "BEGIN{print 7%3}" 
1

```


### 2.7 循环计数

```bash
# 逐行读取/etc/passwd的值，匹配含有bash字段的行，那么x+1,等全部匹配完，利用END只打印最终结果的值。
[root@localhost ~]# awk  '/bash/{x++} END{print x}'  /etc/passwd
2
[root@localhost ~]# awk  '/bash/'  /etc/passwd
root:x:0:0:root:/root:/bin/bash
student:x:1000:1000:student:/home/student:/bin/bash
[root@localhost ~]# 
```

```bash
[root@localhost ~]# who |  awk '$1=="root"{x++} END{print x}'
2
```

## 三、awk条件判断
if判断后面如果只有一个动作指令，则花括号{}可省略，如果if判断后面的指令为多条指令则需要使用花括号括起来，多个指令使用分号分隔。

### 3.1 单分支语句
语法
```bash
if(判断条件){
  动作指令;
}

```

```bash
# 查找内存使用率大于0.3的进程
[root@localhost ~]# ps -eo user,pid,pmem,comm  --sort pmem | awk '{if($3>0.1){print}}'
root        1181  0.2 sssd_nss
gdm         2075  0.3 ibus-x11
gdm         2028  0.4 Xwayland
gdm         1880  0.9 gnome-shell
```


### 3.3 多分支语句
```bash
if(判断条件){
    动作指令;
}else if {
    动作指令；
}else{  
    动作指令；
}


```bash
[root@localhost ~]# awk -F : '{if($NF=="/bin/bash"){printf("用户%s可以登录\n",$1);}else if($NF=="/sbin/nologin"){printf("用户%s不可以登录\n",$1);}else{printf("其余用户%s的shell类型如下:%s \n",$1,$NF);} }' /etc/passwd
用户root可以登录
用户bin不可以登录
用户daemon不可以登录
用户adm不可以登录
用户lp不可以登录
其余用户sync的shell类型如下:/bin/sync 
其余用户shutdown的shell类型如下:/sbin/shutdown 
其余用户halt的shell类型如下:/sbin/halt 
用户mail不可以登录
用户operator不可以登录
用户games不可以登录
用户ftp不可以登录
用户nobody不可以登录
用户dbus不可以登录
```

## 四、awk数组与循环
### 定义数组
注意:数组定义时要用等号赋值，且中间用;隔开。
```bash
[root@localhost ~]# awk 'BEGIN{a[0]=1;a[1]=2;a[2]=3;print a[1]}'
2
```
### 4.1 遍历数组
语法：
```bash
for(变量 in 数组名){
    动作指令序列;
}
```

示例：
```bash
[root@localhost ~]# awk 'BEGIN{a[0]=1;a[1]=2;a[2]=3;for(i in a){print i,a[i]}}'
0 1
1 2
2 3
```

判断元素是否在数组中
```bash
[root@localhost ~]# awk 'BEGIN{a[0]=1;a[1]=2;a[2]=3;if( 12 in a){print("在a里面")}else{print("不在a里面")}}'
不在a里面
[root@localhost ~]# awk 'BEGIN{a[0]=1;a[1]=2;a[2]=3;if( 2 in a){print("在a里面")}else{print("不在a里面")}}'
在a里面
```


### 4.2 for循环
采用与c语言一样的语法格式
```bash
for(表达式1;表达式2;表达式3){
    动作指令序列;
}
```

```bash
[root@localhost ~]# awk 'BEGIN{for(i=0;i<10;i++){print(i)}}'
0
1
2
3
4
5
6
7
8
9
```


### 4.3 while循环

```bash
while(判断条件){
    动作指令；
}
```

```bash
[root@localhost ~]# awk 'BEGIN{ i=1; while(i<5){print(i);i++}}'
1
2
3
4
```

### 4.4 中断语句
与shell类似，awk提供了continue，break,exit循环中断语句。

```bash
[root@localhost ~]# awk 'BEGIN{i=1;while(1){print("hello");i++;if(i==5){print("stop");break;}}}'
hello
hello
hello
hello
stop
```


## 五、awk函数
### 5.1 内置I/O函数

#### getline函数
能让awk立刻读取下一行数据（读取下一条记录并复制给$0，并重新设置NF、NR和FNR。）

```shell

# 解决挂载逻辑卷时，分区信息跨行显示的问题。
 df -h | awk '{if(NF==1){print( $3)};if(NF==6){print($4)}}'
```

#### next函数
停止处理当前的记录输入，立刻读取下一条记录会并返回awk程序的第一个模式重新匹配处理数据。   
类似循环语句中的continue,不会执行当次循环的后续语句。    
```bash
# getline
[root@localhost ~]# awk -F : '/root/{getline;print("next line",$0)} {print("normal line")}' /etc/passwd
next line bin:x:1:1:bin:/bin:/sbin/nologin
normal line
normal line
normal line
normal line
normal line
normal line
normal line
normal line
next line games:x:12:100:games:/usr/games:/sbin/nologin
normal line

# next
[root@localhost ~]# awk -F : '/root/{next;print("next line",$0)} {print("normal line")}' /etc/passwd
normal line
normal line
normal line
normal line
normal line
normal line
normal line
normal line
normal line
normal line
normal line
normal line
normal line
normal line

#经比较可以看出，getline会继续后面指令print "next line",而next不会执行后续指令，而是重新匹配
```

#### system(命令)函数
可以直接在awk中调用shell命令，会启动一个新的shell进程执行命令

```bash
# 注意system中也可以使用变量，但是注意不需要引号，最终命令可以把system中的内容当场命令的整体。
[root@localhost ~]# awk '{system(  "echo " $1 "| grep 127"   )}' /etc/hosts
127.0.0.1
```


#### int(exper)函数
可以对小数取整
```bash
[root@localhost ~]# awk 'BEGIN{print(8/3)}'
2.66667
[root@localhost ~]# awk 'BEGIN{print(int(8/3))}'
2
```


#### rand()函数
返回0到1之间的随机值。
```bash
[root@localhost ~]# awk 'BEGIN{for(i=0;i<5;i++){print(int(rand()*100))}}'
92
59
30
57
74
```



#### length([s])函数
可以统计字符串s的长度，如果不指定字符串s则统计$0的长度。   
```bash
[root@localhost ~]# awk 'BEGIN{a="hello";print(length(a))}'
5
```

#### index(字符串1，字符串2)   
返回字符串2在字符串1中的位置。
```bash
[root@localhost ~]# awk 'BEGIN{a="hello";print(index(a,"l"))}'
3
```

#### match(s,r)
根据正则表达式r返回其在字符串s中的坐标位置
```bash
# 返回第二个小写字母
[root@localhost ~]# awk 'BEGIN{print(match("How much","[a-z]"))}'
2
```

 
 #### tolwer(srt)
 可以将字符串转换为小写
 ```bash
 [root@localhost ~]# awk 'BEGIN{print(tolower("Hello"))}'
hello
```

#### toupper(str)
将字符串转为大写
```bash
[root@localhost ~]# awk 'BEGIN{print(toupper("Hello"))}'
HELLO
```

#### split(字符串，数组，分隔符)
将字符串按特定的分隔符切片后存储在数组中，如果没指定分隔符，则使用IFS定义的。

```bash
# 默认分隔符
[root@localhost ~]# awk  'BEGIN{split("hello world",test); print(test[1],test[2])}'
hello world

# 指定分隔符:
[root@localhost ~]# awk 'BEGIN{split("hello:world",test,":"); print(test[1])}'
hello

```

#### gsub(r,s,[,t])
将字符串t中所有与正则表达式r匹配的字符串全部替换为s，如果没有指定字符串t,则默认对$0
```bash
[root@localhost ~]# head -n 1 /etc/passwd | awk '{gsub("[0-9]","**");print($0)}'
root:x:**:**:root:/root:/bin/bash
```

#### sub(r,s,[,t])
与gsub类型，但仅替换第一个匹配的字符串，而不是替换全部。


#### substr(s,i,[,n])
对字符串s进行截取，从第i位开始，截取n个字符串，如果n没有指定则一直截取到字符串s的末尾位置。
```bash
[root@localhost ~]# awk 'BEGIN{print(substr("hello",1,3))}'
hel
```

#### 用户自定义函数
语法：
```bash
function 函数名(参数列表) { 命令序列 }
```

示例:
```bash
[root@localhost ~]# awk 'function max(x,y){if(x>y){print(x);}else{print(y);}} BEGIN{max(7,5)}'
7
```

## 六、常用命令
```bash
# 查看偶数行，NR输出行号。
[root@localhost ~]# cat -n passwd | awk '{if(NR%2==1){print}}'
     1	root:x:0:0:root:/root:/bin/bash
     3	daemon:x:2:2:daemon:/sbin:/sbin/nologin
     5	lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
     7	shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
     9	mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
    11	games:x:12:100:games:/usr/games:/sbin/nologin
    13	nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
    15	systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
    17	tss:x:59:59:Account used for TPM access:/dev/null:/sbin/nologin

# 去除前面的空格
[root@localhost ~]# echo "    false" | awk '{print($NF)}'
false


# 测试用（利用循环来输出第三列到第5列的内容。）
[root@localhost ~]# cat /etc/passwd | awk -v FS=":" 'BEGIN{print("UID   GID    用户的家目录 ")} {i=3;while(i<6){printf("%s  ",$i);i++};print("\n")}'
UID   GID    用户的家目录 
0  0  root  

1  1  bin  

2  2  daemon  

3  4  adm  

4  7  lp  
```



## 七、常用技巧
### 打印每个硬盘的可用容量
```bash
[root@localhost ~]# df -h | awk '{printf("%-20s %-20s \n",$1,$4)}'
Filesystem           Avail                
devtmpfs             7.8G                 
tmpfs                7.9G                 
tmpfs                7.9G                 
tmpfs                7.9G                 
/dev/mapper/rhel-root 28G                  
/dev/mapper/rhel-home 30G                  
/dev/nvme0n1p1       788M                 
tmpfs                1.6G                 
/dev/sr0             0       
```

### 统计/etc/文件总大小
```bash
# 注意此处的匹配/^-/匹配的是文件类型-代表普通文件（所以此处只统计普通文件的大小。）
[root@localhost ~]# ls -l /etc/  | awk '/^-/{sum+=$5} END{print(sum/1024 "M")}'
983.324M
```

### 统计访问httpd服务的各ip次数
```bash
[root@localhost ~]# awk '{ip[$1]++} END{for(i in ip){print(i,ip[i])}}' /var/log/httpd/access_log-20220419 
192.168.182.1 32
192.168.122.121 613
192.168.122.247 5
192.168.182.148 16
```