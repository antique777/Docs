
## 数组排序的三种方式

1. 使用 Comparable 进行排序；
2. 使用 Comparator 进行排序；
3. 如果是 JDK 8 以上的环境，也可以使用 Stream 流进行排序。

### Comparable排序
```java
public class ListSortExample {
    public static void main(String[] args) {
        // 创建并初始化 List
        List<Person> list = new ArrayList<Person>() {{
            add(new Person(1, 30, "北京"));
            add(new Person(2, 20, "西安"));
            add(new Person(3, 40, "上海"));
        }};
        // 使用 Comparable 自定的规则进行排序
        Collections.sort(list);
        // 打印 list 集合
        list.forEach(p -> {
            System.out.println(p);
        });
    }
}

//  以下 set/get/toString 使用的是 lombok 的注解
@Getter
@Setter
@ToString
class Person implements Comparable<Person> {
    private int id;
    private int age;
    private String name;

    public Person(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }

    @Override
    public int compareTo(Person p) {
        return p.getAge() - this.getAge();
    }
}
```
### Comparator排序

1. 新建Comparator比较器

```java
public class ListSortExample2 {
    public static void main(String[] args) {
        // 创建并初始化 List
        List<Person> list = new ArrayList<Person>() {{
            add(new Person(1, 30, "北京"));
            add(new Person(2, 20, "西安"));
            add(new Person(3, 40, "上海"));
        }};
        // 使用 Comparator 比较器排序
        Collections.sort(list, new PersonComparator());
        // 打印 list 集合
        list.forEach(p -> {
            System.out.println(p);
        });
    }
}

//新建 Person 比较器
class PersonComparator implements Comparator<Person> {
    @Override
    public int compare(Person p1, Person p2) {
        return p2.getAge() - p1.getAge();
    }
}

@Getter
@Setter
@ToString
class Person {
    private int id;
    private int age;
    private String name;

    public Person(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }
}
```

2. 匿名内部类实现

```java
public class ListSortExample2 {
    public static void main(String[] args) {
        // 创建并初始化 List
        List<Person> list = new ArrayList<Person>() {{
            add(new Person(1, 30, "北京"));
            add(new Person(2, 20, "西安"));
            add(new Person(3, 40, "上海"));
        }};
        // 使用匿名比较器排序
        Collections.sort(list, new Comparator<Person>() {
            @Override
            public int compare(Person p1, Person p2) {
                return p2.getAge() - p1.getAge();
            }
        });
        // 打印 list 集合
        list.forEach(p -> {
            System.out.println(p);
        });
    }
}

@Getter
@Setter
@ToString
class Person {
    private int id;
    private int age;
    private String name;
    public Person(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }
}
```

### Stream流排序

1. 常规

```java
public class ListSortExample3 {
    public static void main(String[] args) {
        // 创建并初始化 List
        List<Person> list = new ArrayList<Person>() {{
            add(new Person(1, 30, "北京"));
            add(new Person(2, 20, "西安"));
            add(new Person(3, 40, "上海"));
        }};
        // 使用 Stream 排序
        list = list.stream().sorted(Comparator.comparing(Person::getAge).reversed())
                .collect(Collectors.toList());
        // 打印 list 集合
        list.forEach(p -> {
            System.out.println(p);
        });
    }

    @Getter
    @Setter
    @ToString
    static class Person {
        private int id;
        private int age;
        private String name;
        public Person(int id, int age, String name) {
            this.id = id;
            this.age = age;
            this.name = name;
        }
    }
}
```

2. 第1种字段出现null会报异常，解决方案

`nullsFirst` , `nullsLast`

```java
public class ListSortExample4 {
    public static void main(String[] args) {
        // 创建并初始化 List
        List<Person> list = new ArrayList<Person>() {{
            add(new Person(30, "北京"));
            add(new Person(10, "西安"));
            add(new Person(40, "上海"));
            add(new Person(null, "上海"));
        }};
        // 按照[年龄]正序,但年龄中有一个 null 值
        list = list.stream().sorted(Comparator.comparing(Person::getAge,
                Comparator.nullsFirst(Integer::compareTo)))   
                .collect(Collectors.toList());
        // 打印 list 集合
        list.forEach(p -> {
            System.out.println(p);
        });
    }
}
@Getter
@Setter
@ToString
class Person {
    private Integer age;
    private String name;

    public Person(Integer age, String name) {
        this.age = age;
        this.name = name;
    }
}
```