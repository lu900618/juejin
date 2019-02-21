---
title: 前端面试之道 - ES6 知识点及常考面试题
tags: 面试
notebook: 面试
---

# ES6 知识点及常考面试题章节笔记

## 变量提升

```js
var a = 10
var a
console.log(a) // 10
```

```js
console.log(a) // ƒ a() {}
function a() {}
var a = 1
```

```js
var a = 1
let b = 1
const c = 1
console.log(window.b) // undefined
console.log(window. c) // undefined
```

注：**let const 声明的变量未挂载到window上**

## 原型继承和 class 继承

### 组合继承

```js
function Parent (value) {
  this.val = value
}
Parent.prototype.getValue = function () {
  console.log(this.val)
}
function Child (value) {
  Parent.call(this, value)
}
Child.prototype = new Parent()

const child = new Child(1)

child.getValue() // 1
child instanceof Parent // true
```

以上继承的方式核心是在子类的构造函数中通过 `Parent.call(this)` 继承父类的属性，然后改变子类的原型为 `new Parent()` 来继承父类的函数。

这种继承方式优点在于构造函数可以传参，不会与父类引用属性共享，可以复用父类的函数，但是也存在一个缺点就是**在继承父类函数的时候调用了父类构造函数，导致子类的原型上多了不需要的父类属性，存在内存上的浪费**。

### 寄生组合继承

```js
function Parent (value) {
  this.val = value
}
Parent.prototype.getValue = function () {
  console.log(this.val)
}
function Child (value) {
  Parent.call(this, value)
}
Child.prototype = Object.create(Parent.prototype, {
  constructor: {
    value: Child,
    enumerable: false,
    writable: true,
    configurable: true
  }
})

const child = new Child(1)

child.getValue() // 1
child instanceof Parent // true
```

`Object.create()` 介绍
`Object.create(null)` 创建的对象是一个空对象，在该对象上没有继承 `Object.prototype` 原型链上的属性或者方法
`Object.create()` 方法接受两个参数:`Object.create(obj, propertiesObject)`: `obj` :一个对象，应该是新创建的对象的原型; `propertiesObject` ：可选, 该参数对象是一组属性与值，该对象的属性名称将是新创建的对象的属性名称，值是属性描述符（这些属性描述符的结构与 `Object.defineProperties()` 的第二个参数一样）。注意：该参数对象不能是 `undefined`，另外只有该对象中自身拥有的可枚举的属性才有效，也就是说该对象的原型链上属性是无效的。
总之，**使用 `Object.create()` 是将对象继承到 `__proto__` 属性上**.

### Class 继承

```js
class Parent {
  constructor (value) {
    this.val = value
  }
  getValue () {
    console.log(this.val)
  }
}

class Child extends Parent {
  constructor (value) {
    super(value)
    this.val = value
  }
}

let child = new Child(1)
child.getValue() // 1
child instanceof Parent // true
```

## Proxy

`let p = new Proxy(target, handler)`
`target` 表示需要添加代理的对象，`handler`表示自定义对象中的操作。

```js
let onWatch = (obj, setBind, getLogger) => {
  let handler = {
    get(target, property, receiver) {
      getLogger(target, property)
      return Reflect.get(target, property, receiver)
    },
    set(target, property, value, receiver) {
      setBind(value, property)
      return Reflect.set(target, property, value)
    }
  }
  return new Proxy(obj, handler)
}

let obj = { a: 1 }
let p = onWatch(
  obj,
  (v, property) => {
    console.log(`监听到属性${property}改变为${v}`)
  },
  (target, property) => {
    console.log(`'${property}' = ${target[property]}`)
  }
)
p.a = 2 // 监听到属性a改变
p.a // 'a' = 2
```

注：[Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)、[Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)

## map、filter、reduce

```js
['1','2','3'].map(parseInt)
```

`map` 的回调函数接收三个参数：当前元素、索引、原数组

```js
['1','2','3'].map(function(item, i, arr){
  return parseInt(item, i, arr) // parseInt接收两个参数，分别是值、进制，多传的参数无效
})
```

```js
const arr = [1, 2, 3]
const sum = arr.reduce((acc, current) => acc + current, 0)
console.log(sum) // 6

const mapArray = arr.map(value => value * 2)
const reduceArray = arr.reduce((acc, current) => {
  acc.push(current * 2)
  return acc
}, [])
console.log(mapArray, reduceArray) // [2, 4, 6]
```