## 深拷贝和浅拷贝

### 浅拷贝：
##### 指向了同一块内存，所以修改其中任意的值，另一个值都会随之变化，就是浅拷贝。
var m = { a: 10, b: 20 };
var n = m;
n.a = 13;
console.log(m.a); // 13
n和m指向的是同一个堆，对象复制只是复制的对象的引用。

### 深拷贝
```
var m = { a: 10, b: 20 };
var n = { a: m.a, b: m.b };
n.a = 13;
console.log(m.a); // 10
```
##### m对象与n对象虽然所有的值都一样的，但是在堆里面，对应的不是同一个了，这就是深拷贝。深拷贝和浅拷贝不同，深拷贝是彻底复制一个对象，而不是复制对象的引用。



### 深拷贝和浅拷贝的区别
##### 浅拷贝只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存。但深拷贝会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象。

###### 浅拷贝就想是您和您的影子之间的关系 : 你挂了, 你的影子也跟着挂了
###### 深拷贝就像是您的克隆人, 你挂啦, 可你的克隆人还活着



### 浅拷贝的实现方式

#### 浅拷贝:1.可以通过简单的赋值实现
```
function simpleClone(initalObj) {    
  var obj = {};    
  for ( var i in initalObj) {
    obj[i] = initalObj[i];
  }    
  return obj;
}

var obj = {
  a: "hello",
  b:{
    a: "world",
    b: 21
  },
  c:["Bob", "Tom", "Jenny"],
  d:function() {
    alert("hello world");
  }
}
var cloneObj = simpleClone(obj); 
console.log(cloneObj.b); 
console.log(cloneObj.c);
console.log(cloneObj.d);

cloneObj.b.a = "changed";
cloneObj.c = [1, 2, 3];
cloneObj.d = function() { alert("changed"); };
console.log(obj.b);
console.log(obj.c);
console.log(obj.d);
```


#### 浅拷贝:2.Object.assign()实现
##### Object.assign()拷贝的只是属性值，假如源对象的属性值是一个指向对象的引用，它也只拷贝那个引用值。所以Object.assign()只能用于浅拷贝或是合并对象。这是Object.assign()值得注意的地方。
```
var obj = { a: {a: "hello", b: 21} };
var initalObj = Object.assign({}, obj);
initalObj.a.a = "changed";
console.log(obj.a.a); // changed
```

##### 但当对象只有一层的时候，是深拷贝
```
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = Object.assign({}, obj1);
obj2.b = 100;
console.log(obj1); // {a: 10, b: 20, c: 30}
console.log(obj2); // {a: 10, b: 100, c: 30}
```


### 深拷贝的实现方式
#### 深拷贝:1.简单的赋值
```
var m = { a: 10, b: 20 }
var n = {a:m.a,b:m.b};
n.a = 15;
console.log(m); // {a: 10, b: 20}
console.log(n); // {a: 15, b: 20}
```
##### 说明：m对象和n对象虽然所有的值都是一样的，但是在堆里面，对应的不是同一个了。这就是深拷贝。


#### 深拷贝:2.对象只有一层的话可以使用Object.assign()函数实现。
#### 深拷贝:3.使用JSON.parse()和JSON.stringify()
##### JSON.parse()和JSON.stringify()能正确处理的对象只有Number、String、Array等能够被json表示的数据结构，因此函数这种不能被json表示的类型将不能被正确处理。
```
var obj1 = { body: { a: 10 } };
var obj2 = JSON.parse(JSON.stringify(obj1));
obj2.body.a = 20;
console.log(obj1); // { body: { a: 10 } };
console.log(obj2); // { body: { a: 20 } };
console.log(obj1 === obj2); // false
console.log(obj1.body === obj2.body); // false
```

##### 用JSON.stringify把对象转成字符串，再用JSON.parse转成对象。封装成函数：
```
var cloneObj = function (obj) {
  var str, newObj = obj.constructor === Array ? [] : {};
  if (typeof obj !== 'object') return ;
  if (window.JSON) {
    str = JSON.stringify(obj); // 序列化对象
    newObj = JSON.parse(str); // 还原成新对象
  } else {
    for (var i in obj) {
      newObj[i] = typeof obj[i] === 'object'?cloneObj(obj[i]):obj[i];
    }
  }
  return newObj;
};
```


#### 深拷贝:4.递归拷贝
```
function deepClone(initalObj, finalObj) {    
  var obj = finalObj || {};
  for (var i in initalObj) {
    var prop = initalObj[i]; // 避免相互引用对象导致死循环，如initalObj.a = initalObj的情况
    if(prop === obj) {
      continue;
    }
    if (typeof prop === 'object') {
      obj[i] = (prop.constructor === Array) ? [] : {};
      arguments.callee(prop, obj[i]); // 递归
    } else {
      obj[i] = prop;
    }
  }    
  return obj;
}
var str = {};
var obj = { a: {a: "hello", b: 21} };
deepClone(obj, str);
obj.a = {m:1,n:{a:1,b:2}}
console.log(obj); // {m:1,n:{a:1,b:2}}
console.log(str); // { a: {a: "hello", b: 21} }
```

#### 深拷贝:5.Object.create()：使用Object.create()可以达到深拷贝的效果
#### Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__
```
function deepClone(initalObj, finalObj) {    
  var obj = finalObj || {};    
  for (var i in initalObj) {        
    var prop = initalObj[i];        // 避免相互引用对象导致死循环，如initalObj.a = initalObj的情况
    if(prop === obj) {            
      continue;
    }        
    if (typeof prop === 'object') {
      obj[i] = (prop.constructor === Array) ? [] : Object.create(prop);
      console.log('Object.create(prop):', Object.create(prop));
      // Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__
    } else {
      obj[i] = prop;
    }
  }    
  return obj;
}
var str = {};
var obj = { a: {a: "hello", b: 21} };
deepClone(obj, str);
console.log(obj); // { a: {a: "hello", b: 21} }
console.log(str.a.a); // hello
```


#### 深拷贝:6.jQeury的extend()方法
#### jQuery.extend( [deep ], target, object1 [, objectN ] )，其中deep为Boolean类型，如果是true，则进行深拷贝。
```
var $ = require('jquery');
var obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
var obj2 = $.extend(true, {}, obj1);
console.log(obj1.b.f === obj2.b.f); // false
```


#### 深拷贝:7.lodash:热门函数库提供_.cloneDeep用来做Deep Copy
```
var _ = require('lodash');
var obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
var obj2 = _.cloneDeep(obj1);
console.log(obj1.b.f === obj2.b.f); // false
```
##### 性能不错，用起来简单


