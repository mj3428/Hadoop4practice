## MapReduce与传统关系型比较

| |传统的SQL |MapReduce |
| :---| :------ | :------ |
|数据大小|GB|PB|
|数据存取|交互式和批处理|批处理|
|更新|多次读/写|一次写入，多次读取|
|事务|ACID|无|
|结构|写时模式|读时模式|
|完整性|高|低|
|横向扩展|非线性的|线性的|

## MapReduce数据流范例
input | --> map   | --> shuffle | --> reduce    --> output   
cat * | --> map.rb| --> sort    | --> reduce.rb --> output  

