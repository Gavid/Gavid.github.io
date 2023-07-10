### Shell 里的空语句写作  " : "

```bash
#!/bin/bash
for x in {1..10}
do
    #echo $x
    :
done
```

### 常见比较关系在shell脚本中的转义

> - **lt：less than** 小于
> - **le：less than or equal to** 小于等于
> - **eq：equal to** 等于
> - **ne：not equal to** 不等于
> - **ge：greater than or equal to** 大于等于
> - **gt：greater than** 大于

### shell 数组及其遍历的3种方法

```bash
#!/bin/bash
cwd=$(cd $(dirname $0); pwd)
function main()
{
    echo "shell 数组介绍"
    echo "1.读取数组元素值的一般格式，例如:"
    my_array=(A B "C" D)
    echo "第一个元素为: ${my_array[0]}"
    echo "第二个元素为: ${my_array[1]}"
    echo "第三个元素为: ${my_array[2]}"
    echo "第四个元素为: ${my_array[3]}"
    
    echo "2.获取数组中的所有元素: 使用@ 或 * 可以获取数组中的所有元素,例如:"
    my_array=(A B "C" D)
    echo "数组的元素为: ${my_array[*]}"
    echo "数组的元素为: ${my_array[@]}"
  
    echo "3.获取数组的长度: 获取数组长度的方法与获取字符串长度的方法相同,例如:"
    my_array=(A B "C" D)
    echo "数组元素个数为: ${#my_array[*]}"
    echo "数组元素个数为: ${#my_array[@]}"
 
    echo "***************************************"
    echo "shell 数组遍历的3种方法"
    echo "创建一个数组"
    array=( A B C D 1 2 3 4)
    
    echo "1.标准的for循环"
    for(( i=0;i<${#array[@]};i++)) do
    #${#array[@]}获取数组长度用于循环
    echo ${array[i]};
    done;
 
    echo "2.for … in"
    echo "2.1 遍历(不带数组下标)"
    for element in ${array[@]}
    #也可以写成for element in ${array[*]}
    do
    echo $element
    done
 
    echo "2.2 遍历(带数组下标)"
    for i in "${!array[@]}";   
    do 
    printf "%s\t%s\n" "$i" "${array[$i]}"  
    done
   
   echo "3.While循环法"
   i=0  
   while [ $i -lt ${#array[@]} ]  
   #当变量（下标）小于数组长度时进入循环体
   do  
    echo ${array[$i]}  
    #按下标打印数组元素
    let i++  
   done
    
   echo "4.我的示例"
 
   pos=0
   array=( 20200630 20210731 20200831 )
   for element in ${array[@]}
   do
      end_date=$element
      start_date="${element: 0: 6}01"
      let pos++
      echo "序号: echo ${pos}, start_date: ${start_date}, end_date: ${end_date}"
   done
}
```

### shell循环登陆多台机器进行命令执行

```bash
   for i in ${array[@]}; do
        ssh -T $i bash -s < xxx.sh [参数1] [参数2] ……;
    done
```

### Shell中将多行合并成一行的小技巧

```bash
cat data | tr '\n' '|'        # 行间以|来分隔
```

### Shell中使用nohup调用函数

```bash
function exec_main(){
    # ……
    sleep 1;
}
### start invoke ###
echo "$@" | grep -q -- "--nohup" && exec_main "$@" || echo "$@" | grep -q -- "--nohup" || nohup $scriptName "$@" --nohup >> ${logFile} 2>&1 &
```

### shell脚本获取当前脚本的文件名

```bash
# 1. 带后缀名
echo $0
# 2. 不带后缀名
name=${0%\.*}
echo $name
```

### 在本地shell脚本中ssh到远程服务器并执行命令

```bash
ssh -T [远程ip地址] bash -s < *.sh;
```

### Shell脚本中引用另一个脚本文件

```bash
#方法一: 使用点号
. ./subscript.sh
#方法二: 使用source
source ./subscript.sh
# 注意:
# 1.两个点之间，有空格
# 2.两个脚本不在同一目录，要用绝对路径
# 3.为简单起见，通常用第一种方法
```

### Shell判断字符串包含关系的几种方法

```bash
# 1. 利用grep查找
strA="long string"
strB="string"
result=$(echo $strA | grep "${strB}")
if [[ "$result" != "" ]]
then
    echo "包含"
else
    echo "不包含"
fi
# 2.利用字符串运算符 !!!推荐
strA="helloworld"
strB="low"
if [[ $strA =~ $strB ]]
then
    echo "包含"
else
    echo "不包含"
fi
# 3.利用通配符
A="helloworld"
B="low"
if [[ $A == *$B* ]]
then
    echo "包含"
else
    echo "不包含"
fi
# 4.利用case in 语句
thisString="1 2 3 4 5" # 源字符串
searchString="1 2" # 搜索字符串
case $thisString in 
    *"$searchString"*) echo Enemy Spot ;;
    *) echo nope ;;
esac
# 5.利用替换
STRING_A=$1
STRING_B=$2
if [[ ${STRING_A/${STRING_B}//} == $STRING_A ]]
    then
        ## is not substring.
        echo N
        return 0
    else
        ## is substring.
        echo Y
        return 1
    fi
```

### shell脚本传递日期参数的处理

```bash
#!/bin/bash
d1=$(date -d "$1" +%Y-%m-%d" "%H:%M:%S)  //将时间赋值给变量
if (date > "$d1")
then
  echo "小于当前时间"
else
  echo "大于当前时间"
fi
echo "$d1 ,$*"
```

### Shell 中实现字符串切割的几种方法

```bash
# 方法1：利用shell 中 变量 的字符串替换 （https://blog.csdn.net/github_33736971/article/details/53980123）
#!/bin/bash
string="hello,shell,haha"  
array=(${string//,/ })  
for var in ${array[@]}
do
   echo $var
done 

# 方法2：设置分隔符，通过 IFS 变量
#!/bin/bash
string="hello,shell,haha"
OLD_IFS="$IFS"
IFS=","
array=($string)
IFS="$OLD_IFS"
for var in ${array[@]}
do
   echo $var
done
```

## [shell判断文件是否存在](https://www.cnblogs.com/sunyubo/archive/2011/10/17/2282047.html)

```bash
1. shell判断文件,目录是否存在或者具有权限
2. #!/bin/sh
3.
4. myPath="/var/log/httpd/"
5. myFile="/var /log/httpd/access.log"
6.
7. # 这里的-x 参数判断$myPath是否存在并且是否具有可执行权限
8. if [ ! -x "$myPath"]; then
9. mkdir "$myPath"
10. fi
11.
12. # 这里的-d 参数判断$myPath是否存在
13. if [ ! -d "$myPath"]; then
14. mkdir "$myPath"
15. fi
16.
17. # 这里的-f参数判断$myFile是否存在
18. if [ ! -f "$myFile" ]; then
19. touch "$myFile"
20. fi
21.
22. # 其他参数还有-n,-n是判断一个变量是否是否有值
23. if [ ! -n "$myVar" ]; then
24. echo "$myVar is empty"
25. exit 0
26. fi
27.
28. # 两个变量判断是否相等
29. if [ "$var1" = "$var2" ]; then
30. echo '$var1 eq $var2'
31. else
32. echo '$var1 not eq $var2'
33. fi
```

## 
