# Groovy环境搭建

## 搭建步骤

1. 安装好JDK

2. 下载安装groovySDK工具

3. 配置classpath环境变量

   > - 新建GROOVY_HOME
   > - 在Path后追加%GROOVY_HOME%\bin

4.  控制台输出 groovy -version验证是否安装成功

5. IntelliJ IDEA配置groovy，新版本IDEA默认安装了groovy插件，旧版本可能需要手动安装groovy插件

# Groovy语法

## 基础语法

### 变量

#### 变量的类型

- 基本类型：最终也会被编译器转换成对象类型

- 对象类型

#### 变量的定义

- 强类型定义
- def 弱类型定义，类似js中变量定义

### String字符串

- Java中String

#### GString

**常用的三种定义方式： **

```groovy
println "---------------------1----------------------"
// 使用单引号定义，不可变字符串,也可以指明使用String
def singleName = '单引号字符串';
println singleName
println singleName.class

println "---------------------2----------------------"
// 使用双引号定义，可扩展字符串，存在扩展表达式时，类型为GString类型
def doubleName = "双引号字符串，可扩展${2+3}，${singleName}"
println doubleName
println doubleName.class

println "---------------------3----------------------"
// 使用三个单引号定义，带格式字符串
def threeName = '''第一行
第二行
第三行'''
println threeName
println threeName.class
```

**输出结果：**

> ---------------------1----------------------
> 单引号字符串
> class java.lang.String
> ---------------------2----------------------
> 双引号字符串，可扩展5，单引号字符串
> class org.codehaus.groovy.runtime.GStringImpl
> ---------------------3----------------------
> 第一行
> 第二行
> 第三行
> class java.lang.String

#### groovy对字符串的一些扩充方法

**字符串填充：**

- center ： 中间填充
- padLeft ： 左侧填充
- paDRight：右侧填充

**字符串比较：**

- compareTo方法比较
- 操作符比较，如：str1 > str2

**字符访问：**

- 支持类似数组的形式直接访问字符串字符，如str[0]
- 也支持范围访问，如str[0..1]，前闭后闭区间 

**其他常用方法：**

- reverse：字符串倒置
- capitalize ： 首字母大写
- minus： 支持字符串减法，也可以直接使用减号计算
- isNumber ： 判断是否为数字类型字符串

### 逻辑控制

- groovy中switch可以是任意类型的值，如字符串，列表，范围等

- for循环，对范围，List，Map循环

  ```groovy
  // 循环范围
  int sum
  for (i in 0..9){
      sum += i;
  }
  println sum;
  
  // 循环列表
  sum = 0;
  for(i in [1,2,3,4,5,6,7,8,9]){
      sum += i;
  }
  println sum;
  
  // 循环map
  for(i in ["1":"碧落","2":"黄泉","3":"红尘","4":"紫陌"]){
      println i.key + "." + i.value
  }
  ```

- 其他逻辑控制同Java一致

### 闭包

- 格式：{}
- 参数
- 返回值

**闭包定义：**

```groovy
/**************闭包定义*********************/
// 无参闭包定义
def closure = {println "hello groovy"}
// 闭包调用的两种方式
closure()
closure.call()

// 有参闭包定义
def closure1 = {String name -> println "hello ${name}"}
closure1.call("七夜雪")

// 闭包的返回值,闭包一定有一个一个返回值，不写return时返回为null
def closure2 = {String name -> return name.toUpperCase()}
println closure2.call("hello world!")

// 无参的闭包，有一个默认的参数it
def closure3 = {println it}
closure3.call("Hello，七夜雪！")
```

**基本数据类型:**

- upto
- downto
- times

```groovy
/**************闭包结合基本数据类型使用*********************/
int num = 5;
int result = 1;
// upto计算阶乘
1.upto(num, {int i -> return result *= i})
println result;
result = 1
// downto计算阶乘，闭包可以直接写在方法后面作为参数
num.downto(1){
    int i -> return result *= i
}
println result

result = 0;
// times计算累加值
101.times {result += it}
println result
```

**String结合闭包：**

- echo
- find
- findAll
- any
- every
- collect

#### 进阶

##### 闭包关键变量

- this ：代表闭包定义处的类
- owner ： 代表闭包定义处的类或者对象
- delegate ： 代表任意对象，默认与owner一致

##### 闭包委托策略

## 数据结构                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          

### 列表

- 定义列表：def list = [];  def list = [1,2,3,4,5]
- min
- max
- find
- any
- every
- count

### 映射

- 新建 def map = ['key1':"value1"]
- 访问 map.key1, map[key1]
- 添加元素，map[key2] = value2, map.key2=value
- 默认为LinkedHashMap

### 范围

- def range = 1..10
- 范围是一个list

## Groovy高级语法

### JSON操作

- JSONOutput.toJSON : Java Object -> JSON
- JsonSlurper ： JSON --> Object

### xml文件操作

- XmlSluper
- XmlBuilder

### 文件操作

# Gradle

