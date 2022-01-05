

[官方文档](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md)  

[知乎解释](https://zhuanlan.zhihu.com/p/268312718)

## 依赖
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>最新版本</version>
</dependency>
```

## **特别注意**的地方
1. 手写SQL，使用分页插件时，在xml中SQL的最后不要加`;`，会导致`; limit 1,10`报错。
    - 不过这个简单调用接口就会被发现
2. 调用`PageHelper.startPage`方法后，会对接下来的  **第一个**  MyBatis查询方法做分页操作。
    - 调用接口，发现展示所有数据时才会被发现。数据不够10条时很难注意到

## 常规使用

### 1.最常见
```java
    PageHelper.startPage(1, 10);
    List<User> list = userMapper.selectIf(1);
```
和`PageHelper.offsetPage`方法类似  
在 **需要进行分页的Mybatis查询方法** 前调用`PageHelper.startPage`静态方法即可,紧跟在这个方法后的**第一个MyBatis查询方法**会被进行分页
```java
    //获取第1页，10条内容，默认查询总数count
    PageHelper.startPage(1, 10);
    //紧跟着的第一个select方法会被分页
    List<User> list = userMapper.selectIf(1);
    //分页时，实际返回的结果list类型是Page<E>，如果想取出分页信息，需要强制转换为Page<E>，
    //PageInfo包含了非常全面的分页属性


    //后面的不会被分页，除非再次调用PageHelper.startPage
    List<User> list2 = userMapper.selectIf(null);
```

### 2.offset方式
```java
    PageHelper.offsetPage(1, 10);
    List<User> list = userMapper.selectIf(1);
```
和`startPage`方式类似

### 3.参数方法调用
```java
public interface CountryMapper {
    List<User> selectByPageNumSize(
            @Param("user") User user,
            @Param("pageNum") int pageNum, 
            @Param("pageSize") int pageSize);
}

//配置supportMethodsArguments=true
//在代码中直接调用：
List<User> list = userMapper.selectByPageNumSize(user, 1, 10);
```

要想使用该方式，需要配置`supportMethodsArguments`参数为`true`,同时要配置`params`参数  
```xml
<plugins>
    <!-- com.github.pagehelper为PageHelper类所在包名 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
        <property name="supportMethodsArguments" value="true"/>
        <property name="params" value="pageNum=pageNumKey;pageSize=pageSizeKey;"/>
	</plugin>
</plugins>
```
当调用这个方法时，由于同时发现了 pageNumKey 和 pageSizeKey 参数，这个方法就会被分页。params 提供的几个参数都可以这样使用。  
当从 User 中同时发现了 pageNumKey 和 pageSizeKey 参数，这个方法就会被分页  
注意：pageNum 和 pageSize 两个属性**同时存在**才会触发分页操作，在这个前提下，其他的分页参数才会生效。

### 4.参数对象
```java
    //如果 pageNum 和 pageSize 存在于 User 对象中，只要参数有值，也会被分页
    //有如下 User 对象
    public class User {
        //其他fields
        //下面两个参数名和 params 配置的名字一致
        private Integer pageNum;
        private Integer pageSize;
    }
    //存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
    public interface CountryMapper {
        List<User> selectByPageNumSize(User user);
    }
    //当 user 中的 pageNum!= null && pageSize!= null 时，会自动分页
    List<User> list = userMapper.selectByPageNumSize(user);
```

### 5.ISelect接口方式
```java
//jdk6,7用法，创建接口
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});

//jdk8 lambda用法
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(()-> userMapper.selectGroupBy());
```

```java
    //也可以直接返回PageInfo，注意doSelectPageInfo方法和doSelectPage
    pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(new ISelect() {
        @Override
        public void doSelect() {
            userMapper.selectGroupBy();
        }
    });
    //对应的lambda用法
    pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(() -> userMapper.selectGroupBy());
```

```java
    //count查询，返回一个查询语句的count数
    long total = PageHelper.count(new ISelect() {
        @Override
        public void doSelect() {
            userMapper.selectLike(user);
        }
    });
    //lambda
    total = PageHelper.count(()->userMapper.selectLike(user));
```

### 6.RowBounds方式的调用
```java
List<User> list = sqlSession.selectList("x.y.selectIf", null, new RowBounds(0, 10));
```
使用RowBounds参数进行分页时，代码侵入性最小，只是调用了这个参数，没有增加其他内容  
分页插件检测到使用了RowBounds参数时，会对该查询进行物理分页  
//这种情况下也会进行物理分页查询  
List<User> selectAll(RowBounds rowBounds);  
注意： 由于默认情况下的 RowBounds 无法获取查询总数，分页插件提供了一个继承自 RowBounds 的 PageRowBounds，这个对象中增加了 total 属性，执行分页查询后，可以从该属性得到查询总数。

## PageHelper的安全调用
1. 使用`RowBounds`和`PageRowBounds`参数方式是极其安全的
2. 使用参数方式是极其安全的
3. 使用ISelect接口调用时极其安全的
    - ISelect接口方式除了可以保证安全外，还特别实现了将查询转换为简单的count查询方式，这个方法可以将任意的查询方法，变成一个`select count(*)`方法

## PageHelper的不安全调用
`PageHelper`方法使用了静态的`ThreadLocal`参数，分页参数和线程是绑定的  
1. 只要保证在`PageHelper`方法调用后紧跟MyBatis的查询方法，就是安全的。因为`pagehelper`在`finally`代码段中自动清除了`ThreadLocal`存储的对象。

2. 如果代码在进入 Executor 前发生异常，就会导致线程不可用，这属于人为的 Bug（例如接口方法和 XML 中的不匹配，导致找不到 MappedStatement 时）， 这种情况由于线程不可用，也不会导致 ThreadLocal 参数被错误的使用。

但是如果你写出下面这样的代码，就是不安全的用法：
```java
PageHelper.startPage(1, 10);
List<User> list;
if(param1 != null){
    list = userMapper.selectIf(param1);
} else {
    list = new ArrayList<User>();
}
```
这种情况下由于 param1 存在 null 的情况，就会导致 PageHelper 生产了一个分页参数，但是没有被消费，这个参数就会一直保留在这个线程上。当这个线程再次被使用时，就可能导致不该分页的方法去消费这个分页参数，这就产生了莫名其妙的分页。

上面这个代码，应该写成下面这个样子：
```java
List<User> list;
if(param1 != null){
    PageHelper.startPage(1, 10);
    list = userMapper.selectIf(param1);
} else {
    list = new ArrayList<User>();
}
```
这种写法就能保证安全。

如果你对此不放心，你可以手动清理 ThreadLocal 存储的分页参数，可以像下面这样使用：
```java
List<User> list;
if(param1 != null){
    PageHelper.startPage(1, 10);
    try{
        list = userMapper.selectAll();
    } finally {
        PageHelper.clearPage();
    }
} else {
    list = new ArrayList<User>();
}
```
这么写很不好看，而且没有必要。

## 问题
1. `doSelectPageInfo`是什么?
> doSelectPageInfo是PageHelper.startPage()函数返回的默认Page实例内置的函数  
> 该函数可以用以Lambda的形式通过额外的Function来进行查询而不需要再进行多余的PageInfo与List转换  
> 而doSelectPageInfo的参数则是PageHelper内置的Function(ISelect)接口用以达到转换PageInfo的目的

