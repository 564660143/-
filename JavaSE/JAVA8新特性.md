# Lambda表达式格式

- (参数) -> 单行语句;    不需要显式return, lambda表达式能自行判断出返回值
- (参数) -> {多行语句};  如果方法有返回参数的话,需要显式的使用return
- (参数) -> 表达式;
- 在其他的一些支持函数式编程的语言中, Lambda表达式的类型是函数, 但是在Java中, Lambda表达式是对象, 它们必须依赖于一类特别的对象类型-函数式接口(Functional Interface)

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

# 接口默认方法

- JDK8中, 可以给接口增加具体方法, 接口中的非抽象方法需要使用default关键字进行修饰, 如下:

  ```java
  interface MyInterface {
  
      void print();
  	
      // 接口默认方法
      default void defaultMethod(){
          System.out.println("接口默认实现方法...");
      }
  
  }
  ```

  # 集合内部迭代 

  ```java
  List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
  // 内部迭代
  list.forEach(num -> System.out.println(num));
  // 方法引用内部迭代
  list.forEach(System.out :: println);
  ```

# Function 接口

- JDK8新增的函数式接口
- 接口只有一个抽象方法apply, 接受一个T类型参数, 返回一个R类型参数, T, R表示泛型, 可以相同
- 除了一个抽象的apply方法之外, Function存在两个默认的default方法, compose和andThen, 这两个方法都是用来组合不同的Function的
- 这个函数式接口被大量应用于集合以及Stream(流)中

## apply 方法

```java
/**
 * 输入一个T类型的参数
 * 返回一个R类型的值
 */
R apply(T t);
```

## compose 方法

```
/**
 * 接口默认方法
 * 组合两个函数方法, 返回一个新的函数
 * before的apply方法输入一个V类型的参数, 返回一个T类型的参数
 * 当前的Funtion的apply方法是输入一个T类型的参数, 返回一个R类型的参数
 * 两个函数组合之后, 返回一个新的Function, 新的Function输入一个V类型的参数, 返回一个R类型的参数
 * 简单来说, 入参功能 : V -> T, 当前函数功能 : T -> R, 返回函数功能 : V -> R
 */
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    // 非空判断
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}
```

## andThen 方法

```
/**
 * 组合两个函数方法, 返回一个新的函数
 * after的apply方法输入一个R类型的参数, 返回一个V类型的参数
 * 当前的Funtion的apply方法是输入一个T类型的参数, 返回一个R类型的参数
 * 两个函数组合之后, 返回一个新的Function, 新的Function输入一个T类型的参数, 返回一个V类型的参数
 * 简单来说, 入参功能 : R -> V, 当前函数功能 : T -> R, 返回函数功能 : T -> V
 */
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    // 非空判断
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```

## 代码样例

```java
    public static void main(String[] args) {
        Function<Integer, Integer> function1 = value -> value * 2;
        Function<Integer, Integer> function2 = value -> value + 3;
        Function<Integer, Integer> function3 = function1.compose(function2);
        Function<Integer, Integer> function4 = function1.andThen(function2);
        // function3,先执行function2的apply方法再执行function1的apply方法, 所以结果为(2 + 3) * 2 = 10
        System.out.println("function3 : " + function3.apply(2));
        // function4,先执行function1的apply方法再执行function2的apply方法, 所以结果为2 * 2 + 3 = 7
        System.out.println("function4 : " + function4.apply(2));

        Function<Integer, String> function5 = value -> "Hello World " + value;
        Function<String, List<String>> function6 = value -> new ArrayList<>(Arrays.asList(value));
        Function<Long, Integer> function7 = value -> value.intValue();
        // function8 是function6调用compose组合function5
        // function6是String->List<String>, function5是Integer->String, 所以组合之后,Function8是Integer->List<String>
        Function<Integer, List<String>> function8 = function6.compose(function5);
        // function9 是function7调用andThen方法组合function5
        // function7是Long->Integer, function5是Integer->String, 所以组合之后,Function9是Long->String
        Function<Long, String> function9 = function7.andThen(function5);
        // 输出: [Hello World 1]
        System.out.println(function8.apply(1));
        // 输出: Hello World 100
        System.out.println(function9.apply(100L));
     }
```

# BiFunction接口

- JDK8新增函数式接口
- 可以理解为Function的一种扩展, Function接口接收一个参数, 返回一个参数; BiFunction接口接受两个参数, 返回一个参数
- 唯一的抽象方法apply, 接收两个参数, 返回一个参数
- 存在一个默认方法addThen, 由于apply方法接收两个参数, 所以不存在compose方法
- 和Function一样, 在集合与Stream流中均有应用

## apply 方法

```java
    /**
     * 输入两个参数, 类型分别是T和U, 返回一个结果, 类型为R
     * @param t 函数第一个输入参数
     * @param u 第二个输入参数
     * @return 返回结果
     */
    R apply(T t, U u);
```

## andThen 方法

```java
/**
 * 传入一个Function类型,该Function类型输入参数为R, 返回结果类型为V
 * 当前BiFunction的apply函数输入参数类型为T, U; 返回结果类型为R
 * 将两个参数组合之后返回一个新的BiFunction方法, 输入参数是T,U,返回结果类型为V
 * 简单来说就是当前的BiFunction是(T, U) -> R, 传入的Function是R -> V
 * 所以组合后的新的Function是(T, U) -> V, 即把BiFunction的返回结果作为入参再传入Function中
 */
default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t, U u) -> after.apply(apply(t, u));
}
```

BiFunction没有Compose方法是因为BiFunction的apply方法要接收两个参数, 而不管是Function还是BiFunction乃至所有的Java方法, 返回结果都只有一个

## 代码样例

```java
    public static void main(String[] args) {
        BiFunction<Integer, Integer, Integer> biFunction = (num1, num2) -> num1 + num2;
        // 输出 : 5
        System.out.println(biFunction.apply(2, 3));
        Function<Integer,Integer> function = num -> num * num;
        // newBiFunction的apply(num1, num2)方法等于function.apply(biFunction(num1, num2))
        BiFunction<Integer, Integer, Integer> newBiFunction = biFunction.andThen(function);
        // 输出 : 25
        System.out.println(newBiFunction.apply(2, 3));

        BiFunction<Integer, Integer, List<Integer>> biFunction2 = (num1, num2) -> new ArrayList<>(Arrays.asList(num1, num2));
        Function<List<Integer>,String> function2 = list -> list.toString();
        BiFunction<Integer, Integer, String> newBiFunction2 = biFunction2.andThen(function2);
        System.out.println(newBiFunction2.apply(1, 2));
    }
```

# Predicate 接口

- JDK8 提供的函数式接口
- 提供一个抽象方法test, 接受一个参数, 根据这个参数进行一些判断, 返回判断结果 true / false
- 提供几个默认的default方法, and, or, negate 用于进行组合判断

## test 方法

```java
    /**
     * 接收一个参数, 判断这个参数是否匹配某种规则, 匹配成功返回true, 匹配失败则返回false
     */
    boolean test(T t);
```

## and 方法

```java
    /** 
	 * default方法, 接收另外一个Predicate<T>类型参数进行逻辑与操作
	 * 返回一个新的Predicate
	 * Predicate<T> newPredicate = (t) -> this.test(t) && other.test(t);
     * 如果传入的Predicate为空, 会抛出空指针异常
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
```

## or 方法

```java
    /** 
	 * default方法, 接收另外一个Predicate<T>类型参数进行逻辑或操作
	 * 返回一个新的Predicate
	 * Predicate<T> newPredicate = (t) -> this.test(t) || other.test(t);
     * 如果传入的Predicate为空, 会抛出空指针异常
     */
	default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
```

## negate 方法

```java
    /**
	 * default方法, 返回当前Predicate取反操作之后的Predicate
	 * Predicate<T> newPredicate = (t) -> !test(t);
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
```

## isEqual 方法

```java
    /**
     * 接收一个Object targetRef, 返回一个Predicate<T>类型
     * 返回的Predicate的test方法是用来判断传入的参数是否等于targetRef
     * 如Predicate<T> predicate = Predicate.isEqual("七夜雪");
     * 等同于Predicate<T> predicate = t -> "七夜雪".equals(t);
     * 个人感觉这个方法没多大用处
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
```

## API 使用样例

```java
/**
 * @author 七夜雪
 * @date 2019-01-12 10:53
 */
public class PredicateTest {

    // 辅助打印方法
    private static void print(Object obj) {
        System.out.println(obj);
    }

    public static void main(String[] args) {

        Predicate<String> predicate = item -> "七夜雪".equals(item);
        // 1. test 方法测试
        System.out.println("1---> " + predicate.test("七夜雪"));
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));

        // 2. Predicate 返回一个List中的偶数
        // list.stream(), 表示将List作为流进行处理, filter()方法接收一个Predicate, toArray是将流转换成数组
        Object[] result = list.stream().filter(t -> t % 2 == 0).toArray();
        print("2---> " + Arrays.toString(result));

        // 3. 测试Predicate的and方法, 打印list中大于3, 小于6的数字
        Predicate<Integer> predicate1 = t -> t > 3;
        predicate1 = predicate1.and(t -> t < 6);
        result = list.stream().filter(predicate1).toArray();
        print("3---> " + Arrays.toString(result));

        // 4. 测试Predicate的or方法, 打印list中小于3或大于5的数字
        predicate1 = t -> t < 3;
        predicate1 = predicate1.or(t -> t > 5);
        result = list.stream().filter(predicate1).toArray();
        print("4---> " + Arrays.toString(result));

        // 5. 测试Predicate的negate方法, 返回list中大于等于3,小于等于5的数字, 即对场景4取反
        result = list.stream().filter(predicate1.negate()).toArray();
        print("5---> " + Arrays.toString(result));

        // 6. 测试静态方法isEqual方法, 个人感觉这个方法没啥用处
        predicate = Predicate.isEqual("七夜雪");
        print("6---> " + predicate.test("七夜雪"));
        print("6---> " + predicate.test("七夜雪1"));
    }

}
```

# Supplier 接口

- JDK8 新增函数是接口
- 提供一个get方法, 不接收参数, 返回一个参数

```java
    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
```

# Optional

- JDK8 新增, 主要用于解决 NullPointException异常
- 私有构造方法, 不允许通过new 获得一个Optional实例
- 提供一系列静态工厂方法获得Optional对象
- value为空的Optional可以认为是一个空的Optional

## API 简介

### empty 方法

```java
 
    private static final Optional<?> EMPTY = new Optional<>();
	/*
     * 返回一个空的Optional实例, 这里的空是指value不存在
     */
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
```

### of 方法

```java
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

	/**
     * 返回一个Optional对象, Optional对象的value值为传入的value
     * 如果传入的value为null, 则会抛NullPointException异常
     */
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
```

### ofNullable 方法

```java
    /**
     * 返回一个Optional对象, Optional对象的value为传入的value
     * 如果传入的value为空, 则返回一个空的Optional对象
     * otherwise returns an empty {@code Optional}.
     */
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```

### get 方法

```java
    /**
     * 如果当前Optional的value存在的话, 就返回value
     * 如果value为null,就抛出NoSuchElementException异常
     *
     * @see Optional#isPresent()
     */
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
```

### isPresent 方法

```java
    /**
     * 判断当前Optional的value是否存在, 存在就返回true
     * 如果当前Optional的value为null则返回false
     * @return {@code true} if there is a value present, otherwise {@code false}
     */
    public boolean isPresent() {
        return value != null;
    }


```

### ifPresent 方法

```java
    /**
     * 接收一个函数式接口Consumer用于处理操作
     * 如果value存在(不为null), 则调用传入的Consumer的accept方法进行处理
     * 如果value不存在(为null), 则什么事都不做
     * 如果传入的Consumer为空的话, 则会抛出NullPointerException异常
     */
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```

### filter 方法

```java
    /**
     * 接收一个函数式接口Predicate进行条件匹配
     * 如果传入的predicate为空或predicate匹配失败, 则返回一个空的Optional(value为null)
     * 如果传入的predicate匹配成功, 则返回当前的Optional
     */
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```

### map 方法

```java
    /**
     * 接收一个函数式接口Function类型的参数对当前的Optional进行一些处理以及类型转换
     * 如果传入的Function类型参数为空, 就抛出NullPointException异常
     * 如果当前的Optional是一个空的Optional, 就返回一个空的Optional(value为空)
	 * 如果Function接口的apply方法执行之后, 返回的结果为null, 同样返回一个空的Optional
     * 当前的Optional中value类型为T, 传入的Function类型为Function<T, U>
     * 返回的新的Optional中value类型为U
     */
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

### flatMap 方法

```java
    /**
     * 和map方法功能类似, 但是有一些区别, map方法是将T转换成U之后, 再使用Optional.ofNullable方法
     * 获取Optional<U>类型的Optional, flatMap是function的函数完成对T的处理之后, 直接返回一个
     * Optional<U>类型的Optional, 所以如果Function的apply方法返回为空
     * 则会抛出NullPointerException异常
     */
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }
```

### orElse 方法

```java
    /**
     * 如果当前Optional的value存在, 就返回这个value
     * 如果当前Optional的value不存在, 就返回传入的值other
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }
```

### orElseGet 方法

```java
    /**
     * 方法接收一个Supplier<? extends T>参数
     * 如果当前Optional的value存在, 就返回这个value
     * 如果当前Optional的value不存在,则调用other的get方法生成一个
     * 如果other为空, 则抛出NullPointerException异常
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

## API 用法举例

```java
    public static void main(String[] args) {
        // 1. empty 方法
        Optional<String> optional = Optional.empty();
        // 抛出NoSuchElementException
//        System.out.println(optional.get());

        // 2. of 方法 & get方法
        optional = Optional.of("七夜雪");
        System.out.println("2 ---> " + optional.get());

        // 3. isPresent方法
        if (optional.isPresent()) {
            System.out.println("3 ---> " + optional.get());
        }

        // 4. ifPresent, 测试案例3中这种判断方式和现有的先判断是否为null, 再处理是一致的, 更好的一种方式是使用ifPresent
        optional.ifPresent(item -> System.out.println("4 ---> " + item));
        optional = Optional.empty();
        // 这一行因为optional为空, 所以不会执行输出
        optional.ifPresent(item -> System.out.println("4 ---> " + item));

        // 5. filter方法
        optional = Optional.of("七夜雪").filter(item -> "七夜".equals(item));
        System.out.println("5 --->" + optional.isPresent());
        optional = Optional.of("七夜雪").filter(item -> "七夜雪".equals(item));
        System.out.println("5 --->" + optional.isPresent() + ", value : " + optional.get());

        // 6. map 方法
        Optional<Integer> optional1 = Optional.of(1);
        optional = optional1.map(item -> "七夜雪-> " + item);
        System.out.println("6 ---> " + optional.get());
        optional = optional1.map(item -> null);
        System.out.println("6 ---> " + optional.isPresent());
        optional1 = Optional.empty();
        optional = optional1.map(item -> "七夜雪-> " + item);
        System.out.println("6 ---> " + optional.isPresent());

        // 7. flatMap 方法
        optional1 = Optional.of(1);
        optional = optional1.flatMap(item -> Optional.of("七夜雪-> " + item));
        System.out.println("7 ---> " + optional.get());

        // 8. orElse方法
        System.out.println("8 --->" + optional.get());
        optional = Optional.empty();
        System.out.println("8 --->" + optional.orElse("碧落"));

        // 9. orElseGet 方法
        System.out.println("9 --->" + optional.orElseGet(() -> "红尘"));
        optional = Optional.of("七夜雪");
        System.out.println("9 --->" + optional.orElseGet(() -> "红尘"));

        // 10. ofNullable 方法
        optional = Optional.ofNullable(null);
        System.out.println("10 --->" + optional.isPresent());
        optional = Optional.ofNullable("七夜雪");
        System.out.println("10 --->" + optional.isPresent());
    }
```

# BinaryOperator

- JDK8提供函数式接口
- BiFunction的一种特例, BiFunction是接受的两个参数和返回的一个参数,三个参数类型可以各不相同, 而BinaryOperator的三个参数类型是一致的
- 提供了两个默认方法, minBy和maxBy接收一个Comparator作为参数
- 其中minBy方法返回一个取两个元素较小值的BinaryOperator
- maxBy方法返回一个取两个元素较大值的BinaryOperator

# Stream

流由三部分构成

1. 源
2. 零个或多个中间操作
3. 终止操作

流操作分类:

1. 惰性求值, 中间操作
2. 及早求值, 终止操作

流的创建方式:

```java
1.Streams.of()
2.Arrays.stream()
3.list.stream()
```



## 流的特点

- Collection提供了新的stream方法
- 流不存储值, 通过管道的方式获取值
- 本质是函数式的, 对流的操作会生成一个结果, 不过并不会修改底层的数据源, 集合可以作为流的底层数据源
- 流的中间操作都是延迟操作, 很多流操作(过滤, 映射, 排序等)都可以延迟实现,遇到终止操作时, 才会实际执行流的中间操作



## Stream API

### collect 方法

- 流的短路与并发
- 流的分组与分区

- Collector
  - 同一性和结合性分析
  - 复合
- Collectors
- 收集器用法与多级分组分区