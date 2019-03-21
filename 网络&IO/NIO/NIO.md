# NIO API

## ByteBuffer

### 概念

- capacity 容量, 当前buffer容量
- limit 限制, limit之后的位置不能使用
- position 位置,当前指针指向位置
- mark 标记
- mark <=  position <= limit <= capacity

### 常见操作

- buffer.limit()      // 取出limit位置
- buffer.limit(int nl)   //设置新的limit位置
- buffer.position()     // 获取pos
- buffer.position(int)  // 设置新的pos
- buffer.mark()   // mark=pos,当前位置做标记
- buffer.remaining()  //limit-pos空间
- buffer.hasRemaining()  // limit-pos == 0?
- buffer.rewind()  // position=0,mark丢弃,重绕
- buffer.clear()  // pos=0, limit=cap,mark丢弃,清空
- buffer.flip()  // lim=pos,pos=0,mark丢弃
- buffer.reset()  // pos=mark
- buffer.slice() // 切片,从当前缓冲区划出前n个元素重新构造新缓冲区,n等于当前buffer的remaining大小.两个buffer操纵同一byte[],但有各自属性,如limit,pos等







