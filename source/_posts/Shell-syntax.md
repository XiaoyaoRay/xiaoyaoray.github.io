---
title: Shell脚本语法小结
copyright: true
date: 2018-04-27 15:23:51
tags:
- Shell
- Linux
---

### Shell中 if else以及大于、小于、等于逻辑表达式介绍

- **条件测试的表达式：**

  - [ expression ]  括号两端必须要有空格
  - [[ expression ]] 括号两端必须要有空格
<!--more-->
- **整数比较：**

  - -eq 测试两个整数是否相等

  - -ne 测试两个整数是否不等

  - -gt 测试一个数是否大于另一个数

  - -lt 测试一个数是否小于另一个数

  - -ge 大于或等于

  - -le 小于或等于

  - **命令间的逻辑关系**

    - 逻辑与：&&

    ​        第一个条件为假 第二个条件不用在判断，最总结果已经有
            第一个条件为真，第二个条件必须得判断

    - 逻辑或：||

- **字符串比较**

  - == 等于  两边要有空格
  - != 不等
  - \>  大于
  - <  小于

- **文件测试**

  - -z string 测试指定字符是否为空，空着真，非空为假
  - -n string 测试指定字符串是否为不空，空为假 非空为真
  - -e FILE 测试文件是否存在
  - -f file 测试文件是否为普通文件
  - -d file 测试指定路径是否为目录
  - -r file 测试文件对当前用户是否可读
  - -w file 测试文件对当前用户是否可写
  - -x file 测试文件对当前用户是都可执行
  - -z  是否为空  为空则为真
  - -a  是否不空

- **if语法**

  if 判断条件 0为真 其他都为假

  - 单分支if语句

  ```Shell
  if 判断条件；then
      statement1
      statement2
      .......
  fi
  ```

  - 双分支的if语句：

  ```Shell
  if 判断条件；then
      statement1
      statement2
      .....
      else
      statement3
      statement4
  fi
  ```

  **Note:**
  	if语句进行判断是否为空` [ "$name” = "" ] `等同于`[ ! "$name" ]`或者`[ -z "$name" ]`

  ​

- **例子**

  示例脚本（写一段脚本，输入一个测验成绩，根据下面的标准，输出他的评分成绩（A-F）

  A: 90–100

  B: 80–89

  C: 70–79

  D: 60–69

  F: <60 

  ```shell
  #/bin/bash
  #Verson:0.1
  #Auther:lovelace
  #Pragram:This pragram is calculation your grade
  #import an argument
  read -p "Please input your grade:" x
  declare -i x
  #jugemet $x value is none or not
  if [ "$x" == "" ];then
      echo "You don't input your grade...."
      exit 5
  fi
  #jugement the gread level
  if [[ "$x" -ge "90" && "$x" -le "100" ]];then
      echo "Congratulation,Your grade is A."
  elif [[ "$x" -ge "80" && "$x" -le "89" ]];then
      echo "Good,Your grade is B."
  elif [[ "$x" -ge "70" && "$x" -le "79" ]];then
      echo "Ok.Your grade is C."
  elif [[ "$x" -ge "60" && "$x" -le "69" ]];then
      echo "Yeah,Your grade is D."
  elif [[ "$x" -lt "60" ]];then
      echo "Right,Your grade is F."
  else
      echo "Unknow argument...."
  fi
  ```

  ​

### [Shell中for循环的几个常用写法](https://blog.csdn.net/BabyFish13/article/details/52981110)
