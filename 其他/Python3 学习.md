### Python3 学习

#### 1、基础语法

##### 1.1、标识符

- 第一个字符必须是字母表中字母或下划线 _
- 标识符其他部分由字母、数字和下划线组成
- 标识符对大小写敏感

##### 1.2、保留字

- Python 标准库提供 keyword 模块，可输出当前版本所有关键字

```python
>>> import keyword
>>> keyword.kwlist
['False', 'None', 'True', 'and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```

##### 1.3、注释

- 单行注释以 **#** 开头
- 多行注释用多个 **#** 号，还有 **'''** 和 **"""**

##### 1.4、多行语句：

- \
- [], {}, 或 () 中的多行语句，不需 \

##### 1.5、输入&输出

- "\n\n"在结果输出前输出两个新空行

`input("\n\n按下 enter 键后退出。")`

- 默认输出换行，不换行需在变量尾加 end=""

`print( x ); print( x, end=" " )`

- 格式化输出

`print ("我叫 %s 今年 %d 岁!" % ('小明', 10))`

- 格式化符号

| 符  号 | 描述                                 |
| :----- | :----------------------------------- |
| %c     | 格式化字符及其ASCII码                |
| %s     | 格式化字符串                         |
| %d     | 格式化整数                           |
| %u     | 格式化无符号整型                     |
| %o     | 格式化无符号八进制数                 |
| %x     | 格式化无符号十六进制数               |
| %X     | 格式化无符号十六进制数（大写）       |
| %f     | 格式化浮点数字，可指定小数点后的精度 |
| %e     | 用科学计数法格式化浮点数             |
| %E     | 作用同%e，用科学计数法格式化浮点数   |
| %g     | %f和%e的简写                         |
| %G     | %f 和 %E 的简写                      |
| %p     | 用十六进制数格式化变量的地址         |

- 格式化辅助指令

| 符号  | 功能                                                      |
| :---- | :-------------------------------------------------------- |
| *     | 定义宽度或者小数点精度                                    |
| -     | 用做左对齐                                                |
| +     | 在正数前面显示加号( + )                                   |
| <sp>  | 在正数前面显示空格                                        |
| #     | 在八进制数前面显示零('0')，在十六进制前面显示'0x'或者'0X' |
| 0     | 显示的数字前面填充'0'而不是默认的空格                     |
| %     | '%%'输出单一的'%'                                         |
| (var) | 映射变量(字典参数)                                        |
| m.n.  | m 是显示的最小总宽度,n 是小数点后的位数(如果可用的话)     |

##### 1.6、一行多条语句

- ；

##### 1.7、import & from...import

- 整个模块导入： import somemodule
- 从某模块中导入某函数： from somemodule import somefunction
- 从某模块中导入多函数： from somemodule import firstfunc, secondfunc, thirdfunc
- 将某模块全部函数导入： from somemodule import \*

#### 2、基本数据类型

- 变量不需声明，使用前必须赋值，赋值后才被创建，无类型
- 不可变数据：Number、String、Tuple
- 可变数据：List、Dictionary、Set

##### 2.1、多变量赋值

`a = b = c = 1; a, b, c = 1, 2, "runoob"`

##### 2.2、Number（数字）

- type：查询变量所指对象类型，不认为子类是一种父类类型，`type(a)`
- isinstance：认为子类是一种父类类型，`isinstance(a, int)`，返回 true|false
- del：删除对象引用，`del var_a, var_b` 

- 类型

  - int (整数), 如 1
  - bool (布尔), 如 True
  - float (浮点数), 如 1.23、3E-2
  - complex (复数), 如 1 + 2j、 1.1 + 2.2j

- 运算

  ```python
  >>> 2 / 4  # 除法，得到一个浮点数
  0.5
  >>> 2 // 4 # 除法，得到一个整数
  0
  >>> 17 % 3 # 取余
  2
  >>> 2 ** 5 # 乘方
  32
  ```

- 函数

| 函数            | 返回值                                                       |
| :-------------- | :----------------------------------------------------------- |
| abs(x)          | 绝对值                                                       |
| ceil(x)         | 向上取整                                                     |
| cmp(x, y)       | 如果x<y返回-1，如果x==y返回0，如果x>y返回1                   |
| exp(x)          | e的x次幂                                                     |
| fabs(x)         | 绝对值                                                       |
| floor(x)        | 向下取整                                                     |
| log(x)          | math.log(math.e)返回1.0，math.log(100,10)返回2.0             |
| log10(x)        | 以10为基数的x的对数                                          |
| max(x1, x2,...) | 最大值                                                       |
| min(x1, x2,...) | 最小值                                                       |
| modf(x)         | 返回x的整数部分与小数部分，数值符号与x相同，整数部分以浮点型表示 |
| pow(x, y)       | x**y                                                         |
| round(x [,n])   | x四舍五入值，如给出n值，代表舍入到小数点后的位数，保留到离上一位更近的一端 |
| sqrt(x)         | x的平方根                                                    |

- 随机数函数

| 函数                              | 描述                                                         |
| :-------------------------------- | :----------------------------------------------------------- |
| choice(seq)                       | 从序列元素随机挑选一个                                       |
| randrange ([start,] stop [,step]) | 从指定范围内按指定基数递增的集合中获取一个随机数，基数默认为1 |
| random()                          | 随机生成一个实数，在[0,1)范围                                |
| seed([x])                         | 改变随机数生成器种子seed                                     |
| shuffle(lst)                      | 将序列所有元素随机排序                                       |
| uniform(x, y)                     | 随机生成一个实数，在[x,y]范围                                |

- 三角函数

| 函数        | 描述                              |
| :---------- | :-------------------------------- |
| acos(x)     | 返回x的反余弦弧度值               |
| asin(x)     | 返回x的反正弦弧度值               |
| atan(x)     | 返回x的反正切弧度值               |
| atan2(y, x) | 返回给定的X及Y坐标值的反正切值    |
| cos(x)      | 返回x的弧度的余弦值               |
| hypot(x, y) | 返回欧几里德范数sqrt(x\*x + y\*y) |
| sin(x)      | 返回x弧度的正弦值                 |
| tan(x)      | 返回x弧度的正切值                 |
| degrees(x)  | 将弧度转为角度                    |
| radians(x)  | 将角度转为弧度                    |

- 数学常量

| 常量 | 描述 |
| :--- | :--- |
| pi   | π    |
| e    | e    |

##### 2.3、String（字符串）

- 单引号和双引号相同
- 三引号('''或""")可指定多行字符串
- 转义符 \\
- r 让 \ 不转义。如 r"this is a line with \n"
- "this " "is " "string"会被自动转为 this is string
- 字符串可用 + 连在一起，用 * 表示重复
- 字符串索引：从左往右以 0 开始，从右往左以 -1 开始
- 字符串不能改变
- 无单独字符类型，一个字符是长度为 1 的字符串
- 字符串截取的语法：变量[头下标:尾下标:步长]

```python
#!/usr/bin/python3
str='123456789'
 
print(str)                 # 输出字符串
print(str[0:-1])           # 输出第一个到倒数第二个的所有字符
print(str[0])              # 输出字符串第一个字符
print(str[2:5])            # 输出从第三个开始到第五个的字符
print(str[2:])             # 输出从第三个开始后的所有字符
print(str[1:5:2])          # 输出从第二个开始到第五个且每隔一个的字符（步长为2）
print(str * 2)             # 输出字符串两次
print(str + '你好')         # 连接字符串
 
print('hello\nrunoob')      # 使用反斜杠(\)+n转义特殊字符
print(r'hello\nrunoob')     # 在字符串前面添加一个 r，表示原始字符串，不会发生转义
```

- 转义

| 转义字符   | 描述                                 |
| :--------- | :----------------------------------- |
| \\(在行尾) | 续行                                 |
| \\\        | 反斜杠符号                           |
| \\'        | 单引号                               |
| \\"        | 双引号                               |
| \a         | 响铃                                 |
| \b         | 退格                                 |
| \000       | 空                                   |
| \n         | 换行                                 |
| \v         | 纵向制表符                           |
| \t         | 横向制表符                           |
| \r         | 回车，将 \r 后面的内容移到字符串开头 |
| \f         | 换页                                 |
| \yyy       | 八进制数，y 代表0~7字符              |
| \xyy       | 十六进制数，以\x开头，y代表字符      |
| \other     | 其它字符以普通格式输出               |

- 函数
  - `capitalize()`：将字符串的第一个字符转换为大写
  - `center(width, fillchar)`：返回一个指定宽度 width 居中的字符串，fillchar 为填充的字符，默认为空格
  - `count(str, beg= 0,end=len(string))`：返回 str 在 string 里出现的次数，beg、end 指定范围
  - `bytes.decode(encoding="utf-8", errors="strict")`：解码 bytes 对象，其可以由 `str.encode()` 编码返回
  - `encode(encoding='UTF-8', errors='strict')`：以 encoding 指定编码格式编码字符串，出错默认报 ValueError 异常，除非 errors 指定 'ignore' 或 'replace'
  - `endswith(suffix, beg=0, end=len(string))`：检查字符串是否以 obj 结束，beg、end 指定范围，如果是，返回 True，否则返回 False
  - `expandtabs(tabsize=8)`：把字符串的 tab 符号转为空格，默认空格数 8
  - `find(str, beg=0, end=len(string))`：检测 str 是否包含在字符串中，beg、end 指定范围，如果包含返回开始的索引值，否则返回 -1
  - `index(str, beg=0, end=len(string))`：跟 find() —样，不过如果 str 不在字符串中报异常
  - `isalnum()`：如果字符串至少有一个字符且所有字符都是字母或数字返回 True，否则返回 False
  - `isalpha()`：如果字符串至少有一个字符且所有字符都是字母或中文返回 True，否则返回 False
  - `isdigit()`：如果字符串只包含数字返回 True，否则返回 False
  - `islower()`：如果字符串包含至少一个区分大小写的字符，且其都是小写返回 True，否则返回 False
  - `isnumeric()`：如果字符串只包含数字字符返回 True，否则返回 False
  - `isspace()`：如果字符串只包含空白返回 True，否则返回 False
  - `istitle()`：如果字符串是标题化的返回 True，否则返回 False
  - `isupper()`：如果字符串包含至少一个区分大小写的字符，且其都是大写返回 True，否则返回 False
  - `join(seq)`：以指定字符串作为分隔符，将 seq 中所有元素的字符串表示合并为一个新字符串
  - `len(string)`：返回字符串长度
  - `ljust(width[ , fillchar])`：返回一个原字符串左对齐，并使用 fillchar 填充至长度 width 的新字符串，fillchar 默认空格
  - `lower()`：转换字符串所有大写字符为小写
  - `lstrip()`：截掉字符串左边空格或指定字符
  - `maketrans()`：创建字符映射的转换表，对接受两个参数的调用方式，第一个参数是字符串，表示需转换字符，第二个是字符串，表示转换目标
  - `max(str)`：返回字符串 str 最大的字母
  - `min(str)`：返回字符串 str 最小的字母
  - `replace(old, new[, max])`：将字符串的 old 替换成 new，替换不超过 max 次
  - `rfind(str, beg=0, end=len(string))`：类似 find()，不过从右边开始查找
  - `rindex(str, beg=0, end=len(string))`：类似 index()，不过从右边开始
  - `rjust(width[, fillchar])`：返回原字符串右对齐，用 fillchar(默认空格） 填充至长度 width 的新字符串
  - `rstrip()`：删除字符串末尾的空格或指定字符
  - `split(str="", num=string.count(str))`：以 str 为分隔符截取字符串，如果 num 有指定，仅截取 num+1 个子字符
  - `splitlines([keepends])`：按行('\r', '\r\n', '\n')分隔，返回一个包含各行作为元素的列表，如果 keepends 为 False，不包含换行符，否则保留
  - `startswith(substr, beg=0, end=len(string))`：检查字符串是否以指定子字符串 substr 开头，是返回 True，否则返回 False，beg、end指定范围
  - `strip([chars])`：在字符串执行 lstrip()、rstrip()、swapcase()，将字符串大写转为小写，小写转为大写
  - `title()`：返回"标题化"字符串，所有单词以大写开始，其余字母均为小写
  - `translate(table, deletechars="")`：根据 str 给出的表 (包含256个字符) 转换 string 的字符，过滤的字符放到 deletechars 中
  - `upper()`：转换字符串小写字母为大写
  - `zfill(width)`：返回长度为 width 的字符串，原字符串右对齐，前面填充 0
  - `isdecimal()`：检查字符串是否只包含十进制字符，是返回 true，否则返回 false

##### 2.4、List（列表）

- 列表中元素类型可不同
- 列表截取可接收第三个参数，作用是截取步长，负数表示翻转字符串，`list[1:4:2]`，在索引 1 到索引 4 设置步长 2（间隔一个位置）来截取
- \+ 用于组合列表，* 用于重复列表

```python
#!/usr/bin/python3
list = ['abcd', 786 , 2.23, 'runoob', 70.2]
tinylist = [123, 'runoob']

print (list)            # 输出完整列表
print (list[0])         # 输出列表第一个元素
print (list[1:3])       # 从第二个开始输出到第三个元素
print (list[2:])        # 输出从第三个元素开始的所有元素
print (tinylist * 2)    # 输出两次列表
print (list + tinylist) # 连接列表
```

- 函数
  - `len(list)`：列表元素个数
  - `max(list)`：返回列表元素最大值
  - `min(list)`：返回列表元素最小值
  - `list(seq)`：将元组转为列表
- 方法
  - `list.append(obj)`：在列表末尾添加新对象
  - `list.count(obj)`：统计某元素在列表中出现次数
  - `list.extend(seq)`：在列表末尾一次性追加另一序列多个值（用新列表扩展原来列表）
  - `list.index(obj)`：从列表中找出某个值第一个匹配项的索引位置
  - `list.insert(index, obj)`：将对象插入列表
  - `list.pop([index=-1])`：移除列表元素（默认最后一个元素），返回该元素的值
  - `list.remove(obj)`：移除列表中某个值的第一个匹配项
  - `list.reverse()`：翻转列表
  - `list.sort( key=None, reverse=False)`：对原列表排序
  - `list.clear()`：清空列表
  - `list.copy()`：复制列表

##### 2.5、Tuple（元组）

- 元组中元素类型可不同
- \+ 用于组合元组，* 用于重复元组

```python
#!/usr/bin/python3
tuple = ('abcd', 786 , 2.23, 'runoob', 70.2 )
tinytuple = (123, 'runoob')
tup1 = ()    # 空元组
tup2 = (20,) # 一个元素，需在元素后添加逗号

print (tuple)             # 输出完整元组
print (tuple[0])          # 输出元组的第一个元素
print (tuple[1:3])        # 输出从第二个元素开始到第三个元素
print (tuple[2:])         # 输出从第三个元素开始的所有元素
print (tinytuple * 2)     # 输出两次元组
print (tuple + tinytuple) # 连接元组
```

- 函数
  - `len(tuple)`：计算元组元素个数
  - `max(tuple)`：返回元组中元素最大值
  - `min(tuple)`：返回元组中元素最小值
  - `tuple(iterable)`：将可迭代系列转为元组

##### 2.6、Set（集合）

- 创建空集合用 set() 而不是 { }，{ } 用来创建空字典

```python
#!/usr/bin/python3
sites = {'Google', 'Taobao', 'Runoob', 'Facebook', 'Zhihu', 'Baidu'}
sites.add("Me")		#添加元素
sites.update("Me")	#添加元素
sites.remove("Me")	#移除元素，如果元素不存在，则会发生错误
sites.discard("Me")	#移除元素，如果元素不存在，不会发生错误
sites.pop() 		#对集合进行无序排列，将左面第一个元素删除（随机删除一个元素）
len(s)				#计算集合元素个数
sites.clear()		#清空集合

print(sites)   # 输出集合，重复元素自动去掉

# 成员测试
if 'Runoob' in sites :
    print('Runoob 在集合中')
else :
    print('Runoob 不在集合中')

#集合运算
a = set('abracadabra')
b = set('alacazam')

print(a)
print(a - b)     # a 和 b 的差集
print(a | b)     # a 和 b 的并集
print(a & b)     # a 和 b 的交集
print(a ^ b)     # a 和 b 中不同时存在的元素
```

- 方法
  - `add()`：为集合添加元素
  - `clear()`：移除集合中的所有元素
  - `copy()`：拷贝—个集合
  - `difference()`：返回多个集合的差集
  - `difference_update()`：移除集合元素，元素在指定集合中存在
  - `discard()`：删除集合中指定的元素
  - `intersection()`：返回集合的交集
  - `intersection_update()`：返回集合的交集
  - `isdisjoint()`：判断两集合是否包含相同元素，如果没有返回 True，否则返回 False
  - `issubset()`：判断指定集合是否为该方法参数集合的子集
  - `issuperset()`：判断该方法的参数集合是否为指定集合的子集
  - `pop()`：随机移除元素
  - `remove()`：移除指定元素
  - `symmetric_difference()`：返回两集合中不重复的元素集合
  - `symmetric_difference_update()`：移除当前集合中在另一集合相同元素，并将另一个集合中不同元素插入到当前集合中
  - `union()`：返回两个集合的并集
  - `update()`：给集合添加元素

##### 2.7、Dictionary（字典）

- 无序的 键(key) : 值(value) 集合
- 键(key)必须用不可变类型，同一字典中，键(key)必须唯一

```python
#!/usr/bin/python3
dict = {}
dict['one'] = "1 - 菜鸟教程"
dict[2]     = "2 - 菜鸟工具"

tinydict = {'name':'runoob','code':1, 'site':'www.runoob.com'}

print (dict['one'])       # 输出键为 'one' 的值
print (dict[2])           # 输出键为 2 的值
print (tinydict)          # 输出完整的字典
print (tinydict.keys())   # 输出所有键
print (tinydict.values()) # 输出所有值
```

- 函数
  - `len(dict)`：计算字典元素个数，键总数
  - `str(dict)`：输出字典，以可打印字符串表示
  - `type(variable)`：返回输入变量类型
  - `radiansdict.clear()`：删除字典所有元素
  - `radiansdict.copy()`：返回字典浅复制
  - `radiansdict.fromkeys()`：创建新字典，以序列 seq 元素做字典的键，val 为字典所有键对应初始值
  - `radiansdict.get(key, default=None)`：返回指定键的值，如果键不在字典中返回 default 设置的默认值
  - `key in dict`：如果键在字典 dict 里返回 true，否则返回 false
  - `radiansdict.items()`：以列表返回一个视图对象
  - `radiansdict.keys()`：返回一个视图对象
  - `radiansdict.setdefault(key, default=None)`：和 `get()` 类似，但如果键不在字典中会添加键并将值设为 default
  - `radiansdict.update(dict2)`：把字典 dict2 的键值对更新到 dict 里
  - `radiansdict.values()`：返回一个视图对象
  - `pop(key[,default])`：删除字典给定键 key 对应值，返回值为被删除的值，key 值必须给出，否则返回 default 值
  - `popitem()`：随机返回并删除字典最后—对键值

##### 2.8、数据类型转换

| 函数                   | 描述                                       |
| :--------------------- | :----------------------------------------- |
| int(x [,base\])        | 将x转为整数                                |
| float(x)               | 将x转到浮点数                              |
| complex(real [,imag\]) | 创建复数                                   |
| str(x)                 | 将x转为字符串                              |
| repr(x)                | 将x转为表达式字符串                        |
| eval(str)              | 计算字符串中有效Python表达式，返回一个对象 |
| tuple(s)               | 将序列s转为元组                            |
| list(s)                | 将序列s转为列表                            |
| set(s)                 | 转为可变集合                               |
| dict(d)                | 创建字典，d必须是(key, value)元组序列      |
| frozenset(s)           | 转为不可变集合                             |
| chr(x)                 | 将整数转为字符                             |
| ord(x)                 | 将字符转为整数                             |
| hex(x)                 | 将整数转为十六进制字符串                   |
| oct(x)                 | 将整数转为八进制字符串                     |

#### 3、运算符

##### 3.1、位运算符

| 运算符 | 描述                                                 |
| :----- | :--------------------------------------------------- |
| &      | 按位与                                               |
| \|     | 按位或                                               |
| ^      | 按位异或                                             |
| ~      | 按位取反                                             |
| <<     | 左移，运算数各二进位全左移若干位，高位丢弃，低位补 0 |
| >>     | 右移，运算数各二进位全右移若干位                     |

##### 3.2、逻辑运算符

| 运算符 | 表达式  | 描述                                           |
| :----- | :------ | :--------------------------------------------- |
| and    | x and y | 与，如果x为False，返回x的值，否则返回y的计算值 |
| or     | x or y  | 或，如果x是True，返回x的值，否则返回y的计算值  |
| not    | not x   | 非，如果x为True，返回False，否则返回True       |

##### 3.3、成员运算符

| 运算符 | 描述                                              |
| :----- | :------------------------------------------------ |
| in     | 如果在指定序列中找到值返回True，否则返回False     |
| not in | 如果在指定的序列中没找到值返回True，否则返回False |

##### 3.4、身份运算符

-  `id()`：获取对象内存地址
- is与==区别
  - is 判断两变量引用对象是否为同一个
  - == 判断引用变量值是否相等

| 运算符 | 描述                           |
| :----- | :----------------------------- |
| is     | 判断两标识符是否引用自一个对象 |
| is not | 判断两标识符是否引用自不同对象 |

#### 4、条件控制

```python
if condition_1:
    statement_block_1
elif condition_2:
    statement_block_2
else:
    statement_block_3
```

#### 5、循环

```python
#while循环
while condition:
    statements
#使用else语句，while为false时执行
while <expr>:
    <statement(s)>
else:
    <additional_statement(s)>
#for循环
for <variable> in <sequence>:
    <statements>
else:
    <statements>
#range，生成数列，可指定步长
for i in range(5,9,3):
    print(i)
#pass，空语句
```

#### 6、迭代器

```python
>>> list=[1,2,3,4]
>>> it = iter(list)    # 创建迭代器对象
>>> print (next(it))   # 输出迭代器的下一个元素
1
#for遍历
list=[1,2,3,4]
it = iter(list)    # 创建迭代器对象
for x in it:
    print (x, end=" ")
```

#### 7、函数

- 以 def 开头，后接函数标识符名称和 “(参数):”
- return 结束函数，不带表达式相当于返回 None