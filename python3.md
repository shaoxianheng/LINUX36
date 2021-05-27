# python3

## 3.1.2使用print函数()输入

```
[root@node1 data]# cat py1.py
#!/usr/bin/python3
a = 10
b = 6
fp = open(r'/tmp/a.txt','a+')  
print("数字：",6,file = fp) ##数字
print("表达式：",a*b,file = fp) ##表达式
print("字符串：","成功的唯一秘诀--就是坚持最后一分钟。",file = fp) ##输出到文件
fp.close()  ##关闭文件
```

## 3.2注释

### python语法特点

​		一.注释规则

```
注释是指在程序代码中添加的标注性的文字
```

​		1.单行注释

```
# 注释内容
```

​        vim demon.py

```
#!/usr/bin/python3
#
print('''根据身高、体重计算BMI指数''')
#输入身高和体重
height=float(input("请输入您的身高：")) #要求输入身高，单位是米，如1.70
weight=float(input("请输入您的体重：")) #要求输入体重，单位是kg

bmi=weight/(height*height)  #用于计算BMI指数，公式为"BMI=体重/身高的平方"。
print("您的BMI指数："+str(bmi))

if bmi < 18.5:
    print("您的体重过轻 ~@_@~")
if bmi >= 18.5 and bmi < 24.9:
    print("正常范围，注意保持 ^@_@^")
if bmi >= 24.9 and bmi < 29.9:
    print("老铁你过重，*@_@*")
if bmi >=29.9:
    print("老铁你是肥猪，@_@")

```

​        2. 多行注释

```
在python中将包含子在一对三引号('''...''') 或者("""...""")之间，并不属于任何语句的内容认为是多行注释。
```

```
[root@node1 data]# cat demon.py 
#!/usr/bin/python3
'''
@ 版权所有：吉林省明日科技有限公司
@ 文件名：demon.py
@ 文件功能描述：根据身高、体重计算BMI指数
@ 创建日期 Wed Sep 30 23:28:26 CST 2020
@ 创建人 shaoxh
'''
#
print('''根据身高、体重计算BMI指数''')
#输入身高和体重
height=float(input("请输入您的身高：")) #要求输入身高，单位是米，如1.70
weight=float(input("请输入您的体重：")) #要求输入体重，单位是kg

bmi=weight/(height*height)  #用于计算BMI指数，公式为"BMI=体重/身高的平方"。
print("您的BMI指数："+str(bmi))

if bmi < 18.5:
    print("您的体重过轻 ~@_@~")
if bmi >= 18.5 and bmi < 24.9:
    print("正常范围，注意保持 ^@_@^")
if bmi >= 24.9 and bmi < 29.9:
    print("老铁你过重，*@_@*")
if bmi >=29.9:
    print("老铁你是肥猪，@_@")

```

​		3.中文编码声明注释

```
# _*_ coding:utf-8 _*_
# coding:utf-8
```

## 3.3代码缩进

```
代码缩进是指在每一行代码左端空出一定长度的空白，从而可以更加清晰地从外观上看出程序的逻辑结构。
同一个级别的代码块的缩进量必须相同。如果不采用合理的代码缩进，将抛出SyntaxError异常。
```

```
[root@node1 data]# ./demon.py 
  File "./demon.py", line 20
    print("您的体重过轻 ~@_@~")
                         ^
IndentationError: unindent does not match any outer indentation level


```

```
[root@node1 data]# cat demon.py 
#!/usr/bin/python3
'''
@ 版权所有：吉林省明日科技有限公司
@ 文件名：demon.py
@ 文件功能描述：根据身高、体重计算BMI指数
@ 创建日期 Wed Sep 30 23:28:26 CST 2020
@ 创建人 shaoxh
'''
#
print('''根据身高、体重计算BMI指数''')
#输入身高和体重
height=float(input("请输入您的身高：")) #要求输入身高，单位是米，如1.70
weight=float(input("请输入您的体重：")) #要求输入体重，单位是kg

bmi=weight/(height*height)  #用于计算BMI指数，公式为"BMI=体重/身高的平方"。
print("您的BMI指数："+str(bmi))

if bmi < 18.5:
    print("您的体重过轻 ~@_@~")
  print("您的体重过轻 ~@_@~")
if bmi >= 18.5 and bmi < 24.9:
    print("正常范围，注意保持 ^@_@^")
if bmi >= 24.9 and bmi < 29.9:
    print("老铁你过重，*@_@*")
if bmi >=29.9:
    print("老铁你是肥猪，@_@")

```

