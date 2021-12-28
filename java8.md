# Java8特性

## lambda
> 允许把函数作为一个方法的参数<br>
> 一个匿名方法<br>
> `参数 -> 方法体` 是lambda表达式的最基础的语法<br>
> 使用`->`分隔符分隔参数部分和方法体部分。
>  - 参数部分：
>       - 必须和接口中定义的方法的参数保持一致
>  - 方法体部分：
>       - 如果接口中定义的方法有返回值，则在方法结束之前，一定要有结果返回

```java
//打印这个集合的值
List list = Arrays.asList(1,2,3);
//常规写法
list.forEach(i -> {
    System.out.println(i);
});
//方法体就一行代码，可以把大括号去掉
list.forEach(i -> System.out.println());
//更加简洁的写法
list.forEach(System.out::println);
```

```java
@FunctionalInterface
interface Calculate {
   int calculate(int num1, int num2);
}
​
Calculate c = (int num1, int num2) -> { 
   return num1 + num2;
};
```
特征：
    - 可选类型声明
        - 不需要声明参数类型，编译器可以统一识别参数值
    - 可选的参数圆括号
        - 一个参数无需定义圆括号；但多个参数需要定义圆括号
    - 可选的大括号
        - 如果主体包含了一个语句，就不需要使用大括号
    - 可选的返回关键字
        - 如果主体只有一个表达式返回值，则编译器会自动返回值，大括号需要指定表达式返回了一个数值。

### lambda 表达式的语法精简

#### 参数部分的精简
1. 因为在接口的方法中已经定义了参数的类型和数量，因此，在实现是，可以不用写参数的类型是什么，直接写参数的名字即可。
```java
@FunctionalInterface
interface Calculate {
    int calculate(int num1, int num2);
}
Calculate c = (n1, n2) -> {
    return n1 + n2;
};
```

2. 如果方法的参数列表中，有且只有一个参数的时候，小括号也可以忽略的
```java
@FunctionalInterface
interface Calculate {
    int calculate(int num);
}
Calculate c = n -> {
    return n * 2;
};
```

#### 方法体部分的精简

1. 如果方法体中的语句只有一句，则大括号可以省略
```java
@FunctionalInterface
interface Test {
     void show(String msg);
}
Test t = msg -> System.out.println(msg);
```

2. 如果方法体中唯一的一条语句，是一个返回语句的时候，在省略大括号的同时，return也必须省略
```java 
@FunctionalInterface
interface Calculate {
    int calculate(int num);
}
Calculate c = n -> n * 2;
```

## 方法引用

> 提供了非常有用的语法，可以直接引用已有的Java类或对象（实例）的方法或构造器。结婚Lambda使用，方法引用可以是语言的构造更加紧凑简洁，减少冗余代码。<br>
> `::`

> lambda表达式是为了简化接口实现的，因此，在lambda的实现中不要出现过于复杂的代码。太过复杂的代码会导致代码的可读性变差。因此，如果需要较为复杂的实现，需要先自定义一个方法，然后直接调用这个方法即可。<br>
  如果在lambda表达式中要实现的功能已经在其他的方法中实现了，我们没有必要重新实现，直接使用一个已有的方法即可。此时，比较简单的方式就是使用方法引用，用一个已经存在的方法实现指定的接口。

### 普通的方法引用
如果想要对一个接口进行实现的逻辑，在另一个方法中已经实现了，那么可以直接引用到这个已经实现的方法<br>
方法引用时，需要使用符合`::`，在前面写引用的主体，在后面写引用的方法。静态的方法使用类引用，非静态的方法使用对象引用<br>
方法的引用，还对方法的参数和返回值类型有要求，并不是所有的方法都可以作为被引用的方法。只有当一个方法的参数和返回值类型都与接口中定义的方法一致的时候，才可以进行方法引用

> 1. 静态方法使用类引用，非静态方法使用对象引用
> 2. 只能引用参数、返回值与接口中的定义一致的方法

```java
public class FunctionReferences {
    public static void main(String[] args) {
        // 接口的引用
        // 1、使用类，引用一个静态的方法
        CalculateTest test = FunctionTest::calculate;
        // 2、由于calculate2是私有的权限，在类外不能访问，也不能进行方法引用
        // CalculateTest test2 = FunctionTest::calculate2;
        // 3、使用对象引用一个非静态的方法
        CalculateTest test3 = new FunctionTest()::calculate3;
        // 并不是所有的方法都可以进行引用，必须要保证参数、返回值和接口中定义的方法完全一致才可以
        // CalculateTest test4 = FunctionTest::calculate4;
    }

}

class FunctionTest {
    public static int calculate(int a, int b) {
        if (a > b) {
            return a - b;
        }
        return b - a;
    }

    private static int calculate2(int a, int b) {
        return a - b;
    }

    public int calculate3(int x, int y) {
        return x - y;
    }

    public void calculate4(int a) {

    }
}

@FunctionalInterface
interface CalculateTest {
    int calculate(int a, int b);
}
```

### 构造方法的引用
如果一个接口中的方法，是为了获取一个对象的，此时，可以直接引用到类的构造方法。<br>
引用哪一个构造方法，取决于接口中的方法参数的类型和数量。
```java
public class Test {
    public static void main(String[] args) {
        // 1、由于GetPerson中的get方法是无参的，此时引用的是Person类中的无参构造方法
        GetPerson get = Person::new;

        // 2、由于GetPerson2中的get方法是有参的，此时引用的是Person类中的有参构造方法
        GetPerson2 get2 = Person::new;
    }
}

interface GetPerson {
    Person get();
}

interface GetPerson2 {
    Person get(String name, int age);
}

class Person {
    public Person() { }
    public Person(String name, int age) {

    }
}
```

### setter/getter方法的引用
如果一个函数式接口的方法，参数是一个对象，需要返回一个对象的属性值。此时，这个接口的实现，可以使用对象的getter方法引用来实现。
```java 
public class Test {
    public static void main(String[] args) {
        // 1、因为此时的接口方法参数是一个Person，返回值是一个Person的属性
        //    此时就可以使用getter方法的引用实现了
        GetterTest test = Person::getName;
    }
}

interface GetterTest {
    String getName(Person person);
}

class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
## 默认方法
> 在接口里面有了一个实现的方法

## Stream API
> 函数式编程风格

## Date Time API
> 加强对日期与时间的处理

## Optional
> 用来解决空指针异常

## Nashorn , JavaScript引擎
> 允许在JVM上运行特定的JavaScript应用

