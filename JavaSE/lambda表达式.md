# lambda表达式格式

- (参数) -> 单行语句;    不需要显式return, lambda表达式能自行判断出返回值
- (参数) -> {多行语句};  如果方法有返回参数的话,需要显式的使用return
- (参数) -> 表达式;

# 函数式接口

> 对于只包含一个抽象方法的接口, 可以通过lambda表达式创建该接口的对象. 这种接口称为函数式接口
>
> 可以使用*@FunctionalInterface*

**例如使用Lambda表达式来实现Runnable接口:**

```java
		/**
		 * 使用lambda式实现函数式接口时,只能转换成对应的接口对象, 转换成object对象都是错误的
		 * 如下面这条语句编译会报错 : Object obj = () -> System.out.println("使用lambda表达式来实现Runnable接口") ; 
		 * 输出结果: 使用lambda表达式来实现Runnable接口
		 */
		Runnable r1 = () -> System.out.println("使用lambda表达式来实现Runnable接口") ;
		new Thread(r1).start();
```

- 需要注意的时, 如果你的lambda式存在检查期异常, 那么该异常需要在目标接口的抽象方法中进行声明. 如*Runnable r2 = () -> Thread.sleep(100);*这个lambda表达式无法编译通过.

# 方法引用

> 有时候你想传递给其他代码的操作已经有实现的方法了. 就可以使用方法引用

## 方法引用格式

- **引用静态方法 :** 类名称 :: 静态方法名称
- **引用某个对象的方法 :** 实例化对象 :: 普通方法
- **引用特定类型的方法 :** 特定类 :: 普通方法
- **引用构造方法 :** 类名称 :: new

## 方法引用使用样例

**引用静态方法 :** 

```java
// 声明为函数式接口
interface IMethodRef{
	void print(String s);
}

public static void main(String [] args) {
    /**
     * 将IMethodRef中print方法的实现指向System.out.println方法
     * 这里同样的,接口只能有一个抽象方法
     * 输出结果 : Hello World
     */
    IMethodRef ref = System.out :: println;
    ref.print("Hello World");
}
```

**引用某个对象的方法 :**

```java
// 声明为函数式接口
@FunctionalInterface
interface IMethodRef{
	String upper();
}

public static void main(String [] args) {
    /**
	 * "Hello World" : 表示一个String实例
	 *  IMethodRef的upper方法,引用String实例的toUpperCase()方法,等同于"Hello World".toUpperCase()
	 *  输出结果 : HELLO WORLD
	 */
    IMethodRef ref = "Hello World" :: toUpperCase;
    System.out.println(ref.upper()) ;
}
```

**引用特定类型的方法 :** 

```java
@FunctionalInterface
interface IMethodRef{
	int compareTo(BigDecimal big1, BigDecimal big2);
}

public static void main(String [] args) {
	IMethodRef ref = BigDecimal :: compareTo;
	// 等同于调用BigDecimal.ONE.compareTo(BigDecimal.ZERO), 输出结果是1
	System.out.println(ref.compareTo(BigDecimal.ONE, BigDecimal.ZERO)) ;
	// 等同于调用BigDecimal.ZERO.compareTo(BigDecimal.ONE) , 输出结果是-1
	System.out.println(ref.compareTo(BigDecimal.ZERO, BigDecimal.ONE)) ;
	
}
```

**引用构造方法 :**

```java
package lambda;

class Person{
	private String name;
	private int  age;
	
	public Person(String name, int age) {
		this.name = name ;
		this.age = age ;
	}
	@Override
	public String toString() {
		return "Person [name=" + name + ", age=" + age + "]" ;
	}
}

// 声明为函数式接口
@FunctionalInterface
interface IMethodRef<T>{
	T create(String name, int age);
}

public class MethodRefTest2 {
	
	public static void main(String [] args) {
		// 表示IMethodRef的create引用Person的构造方法来实现
		IMethodRef<Person> ref = Person :: new;
		Person p = ref.create("七夜雪", 18);
		// 输出结果, Person [name=七夜雪, age=18]
		System.out.println(p) ;
	}
	
}

```

