[toc]

---

# Python

## 函数定义和调用

### 使用def语句定义函数

def 函数名（参数列表）:

```python
函数体|语句
```
例：1+1/2+1/3+.....1/n
```python
def a(n):  
  sum = 0.0  
  for i in range(n):  
		sum += 1.0/(i+1)  
	return sum
```
## 类的定义

类是一个数据结构。
```python
class 类名：  
    类体
```
### 对象创建和调用
对象名 = 类名（参数列表）
对象.属性
对象.方法

java 代码：
```java
class person{
    String name;
    int age;
    public person(String name,int age){
        this.name = name;
        this.age = age;
    }
}
```
python 代码:
```python
class Person:
	def __init__(self,name,age):(两个下划线)
		self.name = name
		self.age = age
	def get_info():
	        print(age,name)
class person:
    __age = 45(java中 private static)
    def __init__(self,name):
            self.__name = name(java. private)
    @property
    def name(self):
        return self.__name
```
### 调用私有属性

```python
对象._类名__（两个下划线）私有属性
```
## 属性管理
```python
__getattr__
__getattribute__
__setattr__
__detlattr__
```
## 方法

### 实例方法

def 方法名称（self,【参数列表】）：
### 静态方法
@staticmethod
def 方法名称（【参数列表】）：

### 类方法
@classmethod
def 方法名称（cls,[参数列表]）：

```python
class Methods:
    def publicMethod(self):
        print('公有方法')
    def __privateMethod(self):
        print('私有方法')
    def publicMethod2(self):
        self.__privateMethod()
        
m = Methods()
m.publicMethod()
m.__privateMethod()
m.pulicMethod2()
Methods.public(m)
Methods._Methods__privateMethod(m)
```
## 类的继承

class 子类（父类1，父类2.....）：
 ```python
>>> class person:
	def __init__(self,name,age):
		self.name = name
		self.age = age
	def say_hi(self):
		print("你好，我叫{0},年龄{1}".format(self.name,self.age))

		
>>> class student(person):
	def __init__(self,name,age,stu_id):
		person.__init__(self,name,age)
		self.stu_id = stu_id
	def say_hello(self):
		person.hi(self)
		print("我是学生，我的学号是"，self.stu_id)
 ```
```python
>>> class person:
	def __init__(self,name,age):
		self.name = name
		self.age = age
	def say_hi(self):
		print("你好，我叫{0},年龄{1}".format(self.name,self.age))

>>> class Student(person):
	def __init__(self,name,age,stu_id):
		super(Student,self).__init__(name,age)
		self.stu_id = stu_id
	def say_hello(self):
		super(Student,self).hi(self)
		print('我是学生，我的学号是',self.stu_id)
```
### 查看类的继承关系

```python
类名.mro()
类名.__mro__
```
## 多态
多态：方法的重载与重写，运算符的重载  
```python
>>> class person:
	def __init__(self,name,age):
		self.name = name
		self.age = age
	def __str__(self):
		return "{0},{1}".format(self.name,self.age)

	
>>> p =person('tom',23)
>>> print(p)
tom,23
```
## 文本文件操作

1.打开文件
2.文件操作
3.关闭文件
1）打开文件时，状态为只读模式，默认编码为Unicode
open(文件对象，模式)

```python
open('data1.txt','W')
open('data2.txt', 'X')//存在文件报错
open(file,mode,buffering,encoding,erros,)
```
2)写操作
```python
write()//字符串写进去
writelines()//列表中的字符串写进去
flush()//更新缓存
```
3）读操作

```python
read()
readlines()
readline()
```
4)关闭

```python
f.close()//可以不写
with open('data1.txt','w')as f://既有打开又有关闭
```
例：
```python
with open(r'd:\data1.txt','w',endcoding='utf-8')as f:
	f.write("123\n")
	f.writelines(['abc\n','def\n'])
	

>>> f =open(r'd:\data1.txt','r',endcoding='utf-8')
>>> f.readline()
'123\n'
>>> f.read()
'abc\ndef\n'
>>> f.readlines()
[]
>>> f.close()


>>> with open(r'd:\data1.txt','r',endcoding='utf-8') as f:
	for s in f.readlines():
		print(s,end ="")
```
### 内存文件操作
io模块里提供了两个对象来实现文本文件和二进制文件在内存里的操作：StringIO和BytesIO  
```python
>>> from io import StringIO
>>> f = StringIO('hello\nhi\ngoodbye')
>>> for s in f:
	print(s)

	
hello

hi

goodbye



>>> from io import BytesIO
>>> f = BytesIO()
>>> f.write('中国'.encode('utf-8'))
6
>>> f.seek(0)
0
>>> s = f.read()
>>> print(s)
b'\xe4\xb8\xad\xe5\x9b\xbd'
>>>print(f.getvalue())
```
CSV:逗号分隔符文本模式，用来Excel和数据库进行数据的导入和到处操作
读数据需要使用csv.reader对象
写入数据需要使用的是csv.writer对象
writerow()//写入一行数据
writerows()//写入多行数据
dialect:只读

```python
>>> import csv
>>> def readcsv1(csvfilepate):
	with open(csvfilepath,newline="")as f:
		f_csv = csv.reader(f)
		headers = next(f_csv)
		print(headers)
		for row in f_csv:
			print(row)

>>> if __name__ == '__main__':
	read(路径)
	
import csv
def writecsv(csvfilepath):
   headers = ['学号','姓名','性别','班级']
   rows = [('20180101','张三','男','软件1班'),('20180101','张三','男','软件1班'),('20180101','张三','男','软件1班')]
   with open(csvfilepath,'w',newline='')as f:
   f_csv = csv.writer(f)
```
## 删除文件
remove()
exists()
path

```python
import os
filename  = r'd:\data.txt'
isExists = os.path.exists(filename)
if isExists:
    os.remove(filename)
    print('文件已经成功删除')
else:
    print('要删除的文件不存在')
```
## 文件夹的创建与删除
mkdir():创建一级文件夹
makedirs():创建多级文件夹

```python
import os
dirname = 'd:\pythontest'
multipledirname = r"d:\pythontest1\program\test"
isExists = os.path.exists(dirname)
if isExists:
    print('dirname',文件夹已经存在)
else:
    os.mkdir(dirname)
    print('已经成功创建一个文件夹')
isExists = os.path.exists(multipledirname)
if isExists:
    print('dirname',文件夹已经存在)
else:
    os.makedirs(multipledirname)
    print('已经成功创建一个文件夹')
```
删除文件夹
rmdir():只能删除空文件夹
如果想要删除非空文件夹，shutil中的rmtree()
listdir():返回指定目录下的所有文件和文件夹  

## 对象序列化(对象序列化、串行化)
pickle  
## 异常处理
Python 程序设计中常见错误类型：语法错误，运行时错误，逻辑错误
语法错误：代码拼写错误，也称为编译错误：SyntaxError
运行时错误：在解释执行的过程中产生的错误
逻辑错误：程序可以正常运行，但结果错误
异常处理机制：

```python
try:
    可能产生异常的语句
except:
    捕获异常
else:
    没有发生异常时执行的代码
finally:
    一定会执行
1.try...except...else...finally
2.try...except...else...
3.try...except...
4.try...finally...
```
异常最大父类：
BaseException
子类：
Exception SytemExit KeyboardInterrupt GeneratorExit
1) NameErro:尝试访问没有声明的变量
2) SystaxError:语法错误
3) AttributeError:尝试访问未知对象属性
4) TypeError:类型错误
5) VauleError:数值错误
6) ZeroDivisionError:零除错误
7) IndexError:索引超出边界
8) KeyError:字典关键字不存在  

raise:(等价于java中的throw)
except中子类放到前边，父类放到后边  

### 自定义异常类：   
```python
自定义异常类都要继承Exception或者他的子类
自定义异常类的命名一般都是以Error或者Exception为后缀  
```
### 断言
断言的一般形式有：
    1.前置条件断言
    2.后置条件断言
    3.前后不变断言
assert 布尔类型的表达式
assert 布尔类型的表达式，字符串表达式

### python解释器
```python
Python解释器的运行模式有两种：调试模式和优化模式通常调试模式，内置只读变量__debug__的值为true,但如果使用选项-O运行时，模式变为优化模式__debug__的值为false
```
```python
if__debug__:
    if not testexpression:raise AssertionError

if__debug__:
  if not testexpression:raise AssertionError if__debug__:
     if not testexpression:raise AssertionError
```
### 改进断言
改进方式：采用开源的测试框架进行断言操作  
1. py.test：轻量级的测试框架

    pip install -U pytest 安装
    pytest --version查看版本  
2. unittest:python自带的单元测试框架
    使用该测试框架的时候，不推荐使用assert语句，而是使用的是断言方法self.assertXXX()
3. ptest测试框架 作者是karl
```python
pip install ptest(cmd安装指令)
python2:  
ptest -t 文件名(.类名.方法名)
python3:
ptest3 -t 文件名(.类名.方法名)
```
4. assertpy包  
```python
pip install assertpy
```
```   python
def test():
    assert_that(1+2).is_equal_to(333)
    assert_that('foobars').is_length(7).stars_with('foo').ends_with('bars')
    assert_that(['a','b','c']).contains('a').does_not_contains('x')
if__name__=='__main__':
    test()
```
---
## 数据库
DBMS用于管理和操作数据库的计算机软件系统。可以用于数据的定义检索等操作。
DDL:
DML:
DBMS的分类：
1）适合于企业用户的网络版DBMS：Oracle、SQL Server、IB2、MySQL等. 
2）适合于个人用户的桌面版DBMS：Access。MySQL5.0
数据库模型：关系型、层次型、网状型、面向对象型。
关系型：实体和联系。
数据库中的实体：表、视图、序列、同义词等等
表中的行表表示一个实体的实例
表中的列表表示一个实体的一个特征或属性。
联系:
一对一、一对多、多对多
表中的行（记录或元组）和列（字段）。
通用的数据库访问模块 

1. ODBC
    (1)ODBCInterface
    (2)pyodbc
    (3)mxODBC
2. JDBC
    zxJDBC

## SQLite 数据库
SQLite数据库支持的数据类型：NULL INTEGER REAL TEXT BLOB
Python对应的数据类型:None int float str bytes
要使用的是sqlite3模块，包含的内容： 

```python
sqlite3 Version()
sqlite3 connect()
sqlite3 Connect 数据库连接对象
sqlite3 Cursor 游标对象
sqlite3 Row 行对象
```
步骤：  
1. 导入相应的数据库模块
import sqlite3
2. 建立数据库连接
con = sqlite3.connection()  
3. 创建游标对象
cur = con.cursor()
4. 使用execute()执行sql语句
cur.execute()
5. 结果
6. 数据库提交或回滚
con.commit()
con.rollback()
7. 关闭
con.close()

self.MyFrame.Hiden(True)