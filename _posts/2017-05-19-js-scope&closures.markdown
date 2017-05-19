---
layout:     post
title:      "《YOU DON'T KNOW JS》读书笔记--作用域和闭包"
subtitle:   "读书笔记， js基础"
date:       2017-05-19
author:     "Gavin"
header-img: "img/posts/js/ydkjs-1.jpg"
tags:
    - 前端开发
    - JavaScript
    - 读书笔记
---



## 前言

> 来看看 '你不知道的JS'

javascript 作为前端开发的基础知识，在我们日常研发工作中起着至关重的作用。然而，有多少人对这门编程语言有足够的认知，能够深入理解语言的内部机制，真正发挥'神兵利器'作用呢？让我们跟着KYLE SIMPSON,去探索javascript不为人知的一面


---


## 导图
![](/img/posts/js/scope.png)

## Catalog


1.  [什么是作用域／词法作用域](#什么是作用域／词法作用域)
3.  [函数作用域和块作用域](#函数作用域和块作用域)
4.  [提升](#提升)
5.  [作用域闭包](#作用域闭包)



## 什么是作用域／词法作用域

> 作用域其实就是存储变量和查找变量的规则

javascript事实上是一门编译语言，要想全面的理解作用域是什么，就要知道编译语言的编译原理。
编译流程可以简单的分为三个步骤:
1. 分词／词法分析(Tokenizing/Lexing)
就是把代码分解成词法单元。例如，var a = 2; 会被分解成var、 a、 =、 2、;。
2. 解析/语法分析
把词法单元转换成抽象语法树(AST)
3. 代码生成
生成可执行代码

在分词／词法步骤完成了词法化的工作，这个过程中词法作用域就被确认下来了（除了一些欺骗词法作用域的方法）。
**词法分析阶段，基本能够知道全部标识符在哪里以纪实如何声明的，从而能够预测在执行过程中如果对他们进行查询。**作用域是一种逐级完全嵌套的结构，最外层为全局作用域。代码的执行过程中，对变量的查询会从最内层（当前作用域）开始，逐层向外一直到全局作用域，在碰到第一个匹配的标识符符号时停止（遮蔽效应）。查询主要分为两种类型:
1. LHS 左侧查询，对赋值操作左侧，也就是赋值的目标对象的查询
2. RHS 右侧查询，对赋值操作右侧（不一定在赋值符号右侧，因为赋值的方式有多种），也就是赋值的源头对象的查询。

*  LHS查询如果一直到全局作用域都没有查到匹配对象，在非严格模式下，会创建一个全局变量，严格模式下则会抛出ReferenceError
*  RHS查询如果一直到全局作用域都没有查到匹配对象，会抛出ReferenceError，如果对RHS查询到的对象进行不合理的操作（对非函数进行函数调用，🚰引用null或者undefined类型的值的属性）则会抛出TypeError

###### 欺骗词法作用域的方法:
javaScript 中有两种方法可以用来欺骗词法作用域
1. eval函数，接收一个字符串作为参数，并将其内容视作书写时就存在于程序中这个位置的代码
2. with关键字,根据你传给他的对象凭空创建了一个全新的词法作用域
    ```
    var obj = {
        a:1,
        b:2,
        c
    }
    with(obj){
        a = 3;
        b = 4;
        c = 5;
    }
    ```
  
      
**不推荐使用eval(...) 和 with**
1. 在严格模式下，with被完全禁止，间接或者非安全使用eval(...)也被禁止
2. 因为欺骗了词法作用域，所以在词法分析段的优化是无效的，会对程序性能造成影响


## 函数作用域和块作用域
作用域主要有函数作用域和块作用域两种：
###### 函数作用域
1. 每个函数都会创建一个作用域
2. 利用函数作用域可以规避冲突（避免污染全局命名，实现模块管理）
3. 函数的声明方式主要有函数声明和函数表达式两种
>区分函数声明和函数表达式的区别主要看function关键字是否出现在声明中的第一个词。他们之间最重要的区别是他们的名称标识符绑定在不同的位置。函数声明的名称标识符绑定在其所在作用域，而函数表达式则绑定在自身的函数中
```
//函数声明
function foo(){
}
//函数表达式
(function foo(){
})()
```
4. 表达式可以匿名,但建议使用具名表达式
- **匿名表达式在栈追踪中不会显示出有意义的函数名**
- 匿名函数只能通过argument.callee（已过期）的方式调用自身
- 具名函数的可读性/可理解性更强，不言自明
5. 表达式可以立即执行（IIFE）
- 可以把它们当作函数调用并传递参数进去
- 可以解决undefined被错误覆盖的情况
```
undefined = true;
(function IIFE(undefined){
    var a;
    if (a === undefined){
        console.log("Undefined is safe here")
    }
})
```
- 可以倒置代码运行顺序
```
var a = 2;
(function IIFE(def){
    def( window );
})(function def( global ){
    var a = 3;
    console.log( a );
    console.log( global.a );//2
})
```

###### 块作用域
JavaScript中存在的块作用域主要有如下几种:
1. with
2. try/catch babel等编译器就是使用的try/catch 将ES6中的块作用域语法转换成ES5可运行的代码
3. let/const ES6语法，可以实现块级作用域， const 是常量不可变更

## 提升
JavaScript中，代码并非完全遵循自上而下的方式按序运行，引擎在编译阶段中会找到所有的声明，并用合适的作用域将它们关联起来，所以包括变量和函数在内的所有声明都会在任何代码被执行优先限被处理，于是就有了变量提升的现象。let\和const不会在块级作用域中被提升。
```
a = 2;
var a;
console.log(a); //输出2
```
```
console.log(a); //输出undefined
var a = 2;

```
```
function foo(){
  console.log(a);//undefined
  var a = 2;
  
}
foo()
```
```
foo() //TypeError  foo===undifined
var food = function(){
  console.log(a);
  var a = 2;
}
```

## 作用域闭包

闭包这个概念长期以来一直很难被真正理解，但是如果能够通过上述对作用域的描述，充分理解作用域，那只需要知道闭包的定义，就能很容易的理解闭包。定义如下：
>当函数可以记住并访问所在的词法作用域，即使函数是在当前词法作用域之外执行，这时就产生了闭包

```
function foo() {
    var a = 2;
    function bar() {
        console.log( a );
    }
    return bar;
}
var ba = foo();
baz(); // 2  --这就是闭包
```
######  循环和闭包

```
for (var i=1; i<=5; i++) {
    setTimeout( function timer() {
        console.log( i ); // 输出5次6 不符合语义所暗示的结果
    }, i*1000)
}
```
通过在循环中增加一个闭包可以解决
```
for (var i=1; i<=5; i++) {
   (function(j) {
       setTimeout(function timer() {
           console.log( j ); 
       }, j*1000)
    })(i)
}
```
或者使用let让for循环产生块级作用域
```
for (let i=1; i<=5; i++) {
    setTimeout( function timer() {
        console.log( i ); 
    }, i*1000)
}
```

闭包可以用来实现模块机制
```
function CoolModule(){
    var something = "cool";
    var another = [1, 2, 3];
    function doSomething() {
        console.log(something);
    }
    function doAnother() {
        console.log(another)
    }
    return {
        doSomething: doSomething,
        doAnother: doAnother
    }
}
```
```
var MyModules = (function Manager() {
    var modules = {};
    function define(name, deps, impl) {
        for(var i=0; i<deps.length; i++){
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply( impl, deps );
    }
    function get(name) {
        return modules[name]
    }
    return {
        define: define,
        get:get
    }
})

MyModules.define( "bar", [], function(){
      function hello(who) {
          console.log("let me introduce:" + who)
      }
      return {
          hello: hello
      }
})

MyModules.define( "foo", ["bar"], function(bar){
     var hungry = "hippo";
     function awesome(){
        console.log(bar.hello( hungry ).toUpperCase());
     }
     return {
        awesome:awesome
     } 
})

var bar = MyModules.get('bar');
var foo = MyModules.get('foo');

console.log( bar.hello("hippo") ); //let me introduce: hippo
foo.awesome(); // LET ME INTRODUCE: HIPPO
```
###### 未来的模块机制
ES6中为模块增加了一级语法支持，可以直接通过引用文件作为模块。
对于模块的拓展阅读：
- [CommonJS](http://wiki.commonjs.org/wiki/CommonJS)
- [前端模块化开发那点历史](https://github.com/seajs/seajs/issues/588)
- [AMD (Async Module Definition)](http://wiki.commonjs.org/wiki/Modules/AsynchronousDefinition)
- [SeaJS (CMD) '](http://seajs.org/docs/)

