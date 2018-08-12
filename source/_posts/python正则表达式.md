---
title: python正则表达式
date: 2018-08-12 20:43:27
tags: python
---

# 正则表达式用来匹配字符串中特定的字符
正则表达式简称regex
例如\d是一个正则表达式,表示以为数字字符,即任何一位0-9的数字.

## python中的所有表达式的函数都在re模块中例如

```python
import re

phonNumRegex = re.compile(r'\d\d\d-\d\d\d-\d\d\d\d')

```


# 匹配regex对象

regex对象的search方法传入要查找的字符串,寻找该正则表达式的所有匹配,如果
没有找到则返回none,如果找到了该模式,search方法返回一个match对象,match对象,有一个group方法,他返回被查找字符串中实际匹配的文本

mo = phonNumRegex.search('my number is 455-333-1234')
print(f'phone number is: {mo.group()}')

在python中使用正则表达式有几个步骤:
使用import re导入模块
使用re.complile函数方法创建一个regex对象
想regex对象的search方法传入想要查找的字符串,他返回一个match对象
调用match对象的group方法,返回实际匹配文本的字符串

## 用正则表达式匹配更多模式

### 利用括号分组

假定想要将区号从电话号码中分离,添加括号将在正则表达式中分组:(\d\d\d)-(\d\d\d\-\d\d\d),然后使用group匹配对象方法,冲分组中获取匹配的文本

```python

phonNumRegex = re.compile(r'(\d\d\d)-(\d\d\d-\d\d\d\d)')

mo = phonNumRegex.search('my phone is 444-111-3123')
print(mo.group(1))  # 获得区号
print(mo.group(2))  # 获得电话号码

```

### 如果一次性要获得所有分组需要调用groups()
```python

print(mo.groups())   # 获得所有分组结果为元组
```

利用管道匹配多个分组字符|称为管道,希望匹配许多表达式中的一个时,就可以使它,例如r'Batman|Tina Fey' 将匹配'Batman' 或 'Tina Fey'

```python
heroRegex = re.compile(r'Batman|Tian Fay')
mo1 = heroRegex.search('Batman and Tina Fay')
print(mo1.group())

mo2 = heroRegex.search('Tian fey and Batman')
print(mo2.group())

```

利用findall方法可以找到所有匹配的方法,后面会讲print('*' * 32)使用问好实现可选匹配有时候匹配的模式是可选的,也就是说无论这段文本在不在,正则表达式都会认为匹配字符?标识他前面的分组在这个模式中是可选的,例如

```python
batRegex = re.compile(r'Bat(wo)?man')

mo1 = batRegex.search('the adventures of Batman')
print(mo1.group())

mo2 = batRegex.search('the adventures of Batwoman')
print(mo2.group())
```


### 使用星号进行0次或者多次的匹配

*意味着要进行0次或者多次的匹配,即在*之前进行分组,可以在文本中出现任意次数
也可以完全不存在,或一次又一次的重复例如:
```python

batRegex = re.compile(r'Bat(wo)*man')

mo1 = batRegex.search('the adnentures of Batman')

print(mo1.group())

mo2 = batRegex.search('the adnentures of Batwowowowowowoman')
```

### 使用加号匹配一次或者多次
* 意味着匹配0次或者多次,+则意味着匹配一次或者多次,星号不要求分组出现在匹配
的字符串中,但是加号不同,加号前面的分组必须'至少出现一次'

```python
batRegex = re.compile(r'Bat(wo)+man')

mo1 = batRegex.search('the adventures of Batwoman')



# 如果wo一次也没有出现那么会报错
print(mo1.group())
```
### 使用花括号匹配特定次数

如果想要一个分组特定次数,就在正则表达式中改分组的后面,跟上花括号包裹的数字
例如正则表达式的(Ha){3}将匹配字符串'HaHaHa',但不会匹配'HaHa'因为后者字符串只出现了两次除了一个数字,还可以指定一个范围,即在花括号中写下一个最小值,一个逗号一个最大值例如(Ha){1,3} 将匹配'HaHaHa' 'HaHa' 'Ha',也可以不写花括号中的第一个或者第二个数字,不限定最小值和最大值例如(ha){3,}将匹配三次和更多次数,(ha){,5}将匹配0到5次

```python
haRegex = re.compile(r'(ha){3}')
mo = haRegex.search('hahaha')

print(mo.group())

mo1 = haRegex.search('ha')
print(mo1 == None)

```

### 贪心匹配和非贪心匹配

在字符串'hahaha'中因为(ha){1,3}可以匹配1,2,3个实例,那么为什么字符串
'hahaha'的Match对象调用group()方法会返回'hahaha'而不是'ha'之类的比较短的结果因为python的正则表达式默认是匹配更多的即'贪心'的,这表示在二意的情况去,它会尽可能的匹配最长的字符串,而花括号的非贪心版本匹配尽可能短的字符串,即在结束的花括号后跟着一个问好

```python
greedyHaRegex = re.compile(r'(ha){3,5}?')

mo = greedyHaRegex.search('hahahahaha')

print(mo.group())
```

### findall()方法

除了search方法之外Regex对象还有一个findall方法,search方法将返回一个Match对象包含被查找字符串中第一次匹配的的文本,而findall方法将返回一组字符串,包含被查找字符串中的所有匹配

```python
phonNumRegex = re.compile(r'\d\d\d-\d\d\d-\d\d\d\d')

mo = phonNumRegex.search('call: 515-222-1234 work: 121-333-1231')
print(mo.group())
```


### findall 方法返回一个列表

```python
print(phonNumRegex.findall('call: 515-222-1234 work: 121-333-1231'))
```


- 字符分类
- \d 0-9的任何数字
- \D 除了0-9之外的任意字符
- \w 任何字母,数字或下划线(可以认为是匹配'单词'字符)
- \W 除了字母,数字和下划线以外的任何字符
- \s 空格,制表符或换行符(可以认为匹配'空白'字符)
- \S 除了空格制表符或者换行符以外的任何字符

字符分类对于缩短正则表达式很有用,字符分类[0-5]只匹配数字0-5


### 插入字符
可以在正则表达式的开始处使用插入符号^,标明匹配必须发生在被查找文本的开始处类似的,可以在正则表达式的末尾加上$,表示该字符串必须以这个正则表达式的模式结束可以同时使用^和$

例如使用r'Hello'匹配以'Hello'开始的字符串

```python
beginsWithHello = re.compile(r'^hello')

mo = beginsWithHello.search('hello world')

print(mo.group())

```
beginsWithHello.search('he said hello') 这条是无法匹配的

### 正则表达式r'\d$'匹配以数字0-9结束的字符串
```python

endsWithNumber = re.compile(r'\d$')

print(endsWithNumber.search('you number is 43').group())
```


### 正则表达r'\d+$'表示匹配数字开头和结尾的字符串

```python
wholeStringIsNum = re.compile(r'\d+$')

print(wholeStringIsNum.search('3 is 33').group())

```

### 通配字符

在正则表达式中 .  字符被称为"通配符",它匹配除了换行之外的所有字符

```python
atRegex = re.compile(f'.at')

print(atRegex.findall('the can in the hat sat on the flat mat'))
```


.at 只匹配一个字符这也就是上面的flat 只匹配到flat

### 用.* 匹配所有字符

```python
nameRegex = re.compile(f'First Name:(.*) Last Name:(.*)')

print(nameRegex.search('First Name: AL Last Name: Sweigart').group())

```

### 用句点字符匹配换行

.*将匹配除了换行之外的所有字符,通过传入re.DOTALL作为re.compole的第二个参数
可以让.匹配所有字符,包括换行符
```python

noNewLineRegx = re.compile('.*', re.DOTALL)
print(noNewLineRegx.search('serve the public trust .\n Protect the innocent').group())

```

正则表达式符号复现

-  ?匹配零次或者一次前面的分组
- *匹配零次或者多次前面的分组
- +匹配一次或者多次前面的分组
- {n}匹配 n 次前面的分组
- {n,}匹配 n 次到多次前面的分组
- {,m} 匹配零次到m次前面的分组
- {n,m}匹配至少n次,至多m次前面的分组
- {n,m}?或*?或+?对前面的分组都是进行非贪心算法
- ^spam 意味着字符串必须与spam开始
- spam$ 意味着字符串必须与spam结尾
- .匹配所有字符,除了换行符
- \d,\w,\s 分别匹配数字单词和空格
- \D,\W,\S 分别匹配出数字,单词和空格外的所有字符
- [abc]匹配方括号内的任意字符
- [^abc]匹配不在方括号内的任意字符

### 不区分大小写
不区分大小写可以向compile传入第二个参数re.I或者re.IGNORECASE

```python
pobocop = re.compile(r'robocop', re.I)

print(pobocop.search('RoboCop is ...').group())

```
### 使用sub方法替换字符串
正则表达式不仅能够找到文本模式,而且能够用新的文本替换掉这些模式,Regex对象的sub方法需要传入两个参数,第一个是字符串,用于取代发现的匹配,第二个参数是一个字符串,用于正则表达式匹配的内容,返回替换完成的字符串

```python
nameRegex = re.compile(r'Agent \w+')

print(nameRegex.sub('GENSORED', 'Agent Alice gave the secret document to Agent Bob'))

```