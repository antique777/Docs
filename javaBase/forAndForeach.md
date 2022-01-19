
## 讨论维度

1. for循环和forEach的本质区别。
2. for循环和forEach的语法区别。
3. for循环和forEach的性能区别。

> **本质区别**

for循环是js提出时就有的循环方法。forEach是ES5提出的，挂载在可迭代对象原型上的方法，例如Array Set Map。forEach是一个迭代器，负责遍历可迭代对象。那么遍历，迭代，可迭代对象分别是什么呢。

遍历：指的对数据结构的每一个成员进行有规律的且为一次访问的行为。

迭代：迭代是递归的一种特殊形式，是迭代器提供的一种方法，默认情况下是按照一定顺序逐个访问数据结构成员。迭代也是一种遍历行为。

可迭代对象：ES6中引入了 iterable 类型，Array Set Map String arguments NodeList 都属于 iterable，他们特点就是都拥有 [Symbol.iterator] 方法，包含他的对象被认为是可迭代的 iterable。

在了解这些后就知道 forEach 其实是一个迭代器，他与 for 循环本质上的区别是 forEach 是负责遍历（Array Set Map）可迭代对象的，而 for 循环是一种循环机制，只是能通过它遍历出数组。

再来聊聊究竟什么是迭代器，还记得之前提到的 Generator 生成器，当它被调用时就会生成一个迭代器对象（Iterator Object），它有一个 .next()方法，每次调用返回一个对象{value:value,done:Boolean}，value返回的是 yield 后的返回值，当 yield 结束，done 变为 true，通过不断调用并依次的迭代访问内部的值。

迭代器是一种特殊对象。ES6规范中它的标志是返回对象的 next() 方法，迭代行为判断在 done 之中。在不暴露内部表示的情况下，迭代器实现了遍历。看代码

```java
let arr = [1, 2, 3, 4]  // 可迭代对象
let iterator = arr[Symbol.iterator]()  // 调用 Symbol.iterator 后生成了迭代器对象
console.log(iterator.next()); // {value: 1, done: false}  访问迭代器对象的next方法
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: 4, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```
只要是可迭代对象，调用内部的 Symbol.iterator 都会提供一个迭代器，并根据迭代器返回的next 方法来访问内部，这也是 for...of 的实现原理
```java
let arr = [1, 2, 3, 4]
for (const item of arr) {
    console.log(item); // 1 2 3 4 
}
```

把调用 next 方法返回对象的 value 值并保存在 item 中，直到 value 为 undefined 跳出循环，所有可迭代对象可供for...of消费。再来看看其他可迭代对象：

```java
function num(params) {
    console.log(arguments); // Arguments(6) [1, 2, 3, 4, callee: ƒ, Symbol(Symbol.iterator): ƒ]
    let iterator = arguments[Symbol.iterator]()
    console.log(iterator.next()); // {value: 1, done: false}
    console.log(iterator.next()); // {value: 2, done: false}
    console.log(iterator.next()); // {value: 3, done: false}
    console.log(iterator.next()); // {value: 4, done: false}
    console.log(iterator.next()); // {value: undefined, done: true}
}
num(1, 2, 3, 4)

let set = new Set('1234')
set.forEach(item => {
    console.log(item); // 1 2 3 4
})
let iterator = set[Symbol.iterator]()
console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: 4, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```
所以我们也能很直观的看到可迭代对象中的 Symbol.iterator 属性被调用时都能生成迭代器，而 forEach 也是生成一个迭代器，在内部的回调函数中传递出每个元素的值。

（感兴趣的同学可以搜下 forEach 源码， Array Set Map 实例上都挂载着 forEach ，但网上的答案大多数是通过 length 判断长度， 利用for循环机制实现的。但在 Set Map 上使用会报错，所以我认为是调用的迭代器，不断调用 next，传参到回调函数。由于网上没查到答案也不妄下断言了，有答案的人可以评论区留言）

## for循环和forEach的语法区别

1. forEach 的参数。
2. forEach 的中断。
3. forEach 删除自身元素，index不可被重置。
4. for 循环可以控制循环起点。

### forEach 的参数

```java
arr.forEach((self,index,arr) =>{},this)
```
- self： 数组当前遍历的元素，默认从左往右依次获取数组元素。
- index： 数组当前元素的索引，第一个元素索引为0，依次类推。
- arr： 当前遍历的数组。
- this： 回调函数中this指向。

```java
let arr = [1, 2, 3, 4];
let person = {
    name: '技术直男星辰'
};
arr.forEach(function (self, index, arr) {
    console.log(`当前元素为${self}索引为${index},属于数组${arr}`);
    console.log(this.name+='真帅');
}, person)
```

可以利用 arr 实现数组去重

```java
let arr1 = [1, 2, 1, 3, 1];
let arr2 = [];
arr1.forEach(function (self, index, arr) {
    arr.indexOf(self) === index ? arr2.push(self) : null;
});
console.log(arr2);   // [1,2,3]
```

![](./_media/20220119085330.jpg)

### forEach 的中断
