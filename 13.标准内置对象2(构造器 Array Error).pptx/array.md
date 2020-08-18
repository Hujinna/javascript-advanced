# Array.apply(null, { length: 2 })

## 1. Array构造函数

直接调用 Array 函数跟 new 方式调用是等价的，即:        

```js
var a = Array(2); // 等价于 var a = new Array(2);
```

表示：创建一个长度为 2 的数组，**注意该数组的元素并没有被初始化**，即：

```js
console.log(0 in a); // false
console.log(1 in a); // false, 因为数组下标 0，1 还未初始化
console.log(a[0]); // undefined, 因为数组下标 0 还未初始化,访问不存在的属性返回 undefined
```

## 2. apply函数

ES5 开始 `apply` 函数的第二个参数除了可以是数组外，还可以是**类数组对象**（即包含 length 属性，且 length 属性值是个数字的对象）。对象 `{length: 2}` 就是一个类数组对象，因为没有初始化下标 0，1 的值，所以获取0，1 下标的值得到的都是undefined。

```js
console.log(a[0]); // undefined
console.log(a[1]); // undefined
// 可以转成真正的数组
var a = Array.prototype.slice.call({length: 2});
console.log(Array.isArray(a)) // true
```

## 3. 分析 Array.apply(null, { length: 2 }) 的含义

再看表达式 `Array.apply(null, { length: 2 })`  就等价于：

```js
// 1 熟悉一点: 将 {length: 2} 作为 Array.apply 的第二个参数，相当于 [undefined, undefined] 作为Array.apply 的第二个参数。
Array.apply(null, [undefined, undefined]); 

// 2 再熟悉一点：apply 方法的执行结果等价于
Array(undefined, undefined); 

// 3 再再熟悉一点：Array 方法直接调用和 new 方式调用等价
new Array(undefined, undefined); 
```

这样就很容易知道该表达式的值是**一个长度为 2，且每个元素值都被赋值为 undefined的 数组（注意此时不是数组元素没有初始化，而是初始化成 undefined，这就是跟 Array(2) 的区别）**。

## 4. 需求(目的)

`map` 方法会给原数组中的每个元素都按顺序调用一次  `callback` 函数。`callback` 函数只会在**有值的索引上**被调用；那些从来没被赋过值或者使用 `delete` 删除的索引则不会被调用。

`filter` 为数组中的每个元素调用一次 `callback` 函数。`callback` 只会在**已经赋值的索引上**被调用，对于那些已经被删除或者从未被赋值的索引不会被调用。

......

所以，写 `Array.apply(null, { length: 2 })` 这么复杂的表达式的目的是**创建一个长度为2，且每个元素都被初始化的数组**。

```js
// 被初始化的数组
Array.apply(null, {length: 20}).map(function(val, index){
   console.log(index); // 循环20次
});
// 未被初始化的数组
Array(20).map(function(val, index){
   console.log(index); // 不会被执行
});
```

## 5. 其他

1. 如果为了少写几个字的话还可以把该表达式修改成：

   ```js
   Array.apply(null, Array(20)); // 第二个参数用Array(20)代替{length: 20}
   ```

2. 还可以使用ES6 API更直观表达意图：

   ```js
   // 方法1：
   Array.from({length: 20})
   
   // 方法2
   Array(20).fill(null)
   ```

