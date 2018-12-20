![1538054967411](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1538054967411.png)

Join语句的优化

- 尽可能减少Join语句中的NestedLoop的循环总次数; "永远用小结果集驱动大的结果集"
- 优先优化NestedLoop的内层循环;
- 保证Join语句中被驱动表上的Join条件字段已经被索引