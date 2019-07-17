# Hadoop Streaming
Hadoop提供MapReduce的API，允许你使用非JAVA的其他语言来写自己的map与reduce函数。  
Hadoop Streaming使用Unix标准流作为Hadoop和应用程序之间的接口。  
Streaming天生适合用于文本处理。map的输入数据通过标准输入流传递给map函数，并且是一行一行的传输，最后将结果行写到标准输出。map输出的
键值对是以一个制表符分隔的行，reduce函数的输入格式与之相同并通过标准输入流进行传输。reduce函数从标准输入流中读取输入行，该输入
已由Hadoop框架根据键排过序，最后将结果写入标准输出。  
*Py版本map函数 查找最高气温*  
```
import re
import sys

for line in sys.stdin:
  val = line.strip()
  (year, temp, q) = (val[15:19], val[87:92], val[92:93])
  if (temp != "+9999" and re.match("[01459]", q)):
    print "%s\t%s" % (year, temp)
```

*Py版本reduce函数 查找最高气温*  
```
import sys

for line in sys.stdin:
  (key, val) = line.strip().split("\t")
  if last_key and last_key != key:
    print "%s\t%s" % (last_key, max_val)
    (last_key, max_val) = (key, int(val))
  else:
    (last_key, max_val) = (key, max(max_val, int(val)))
  if last_key:
    print "%s\t%s" % (last_key, max_val)
```
