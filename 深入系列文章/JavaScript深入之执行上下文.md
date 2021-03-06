# JavaScript深入之执行上下文

# 前言

在《JavaScript深入之执行上下文栈》中讲到，当JavaScript代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。

对于每个执行上下文，都有三个重要属性：

* 变量对象(Variable object，VO)
* 作用域链(Scope chain)
* this

然后分别在《JavaScript深入之变量对象》《JavaScript深入之作用域链》《JavaScript深入之从ECMAScript规范解读this》中讲解了这三个属性。

阅读本文前，如果对以上概念不是很清楚，希望先行阅读这些文章。

这一章，我们会结合着所有内容，讲讲执行上下文的具体处理过程。

## 具体执行分析

依然是以《JavaScript深入之执行上下文栈》中权威指南的例子进行讲解，毕竟是说了要具体讲解两个例子之间有什么不同：

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

执行过程如下：

1.执行全局代码，创建全局执行上下文，全局上下文被压入执行上下文栈

```js
    ECStack = [
        globalContext
    ];
```

2.全局上下文初始化

```js
    globalContext = {
        VO: [global, scope, checkscope],
        Scope: [globalContext.VO],
        this: globalContext.VO
    }
```

2.初始化的同时，checkscope函数被创建，保存作用域链到[[scope]]

```js
    checkscope.[[scope]] = [
      globalContext.VO
    ];
```

3.执行checkscope函数，创建checkscope函数执行上下文，checkscope函数执行上下文被压入执行上下文栈

```js
    ECStack = [
        checkscopeContext,
        globalContext
    ];
```

4.checkscope函数执行上下文初始化：

1. 复制函数[[scope]]属性创建作用域链，
2. 用arguments创建活动对象，
3. 初始化活动对象，即加入形参、函数声明、变量声明，
4. 将活动对象压入checkscope作用域链顶端。

同时f函数被创建，保存作用域链到[[scope]]

```js
    checkscopeContext = {
        AO: {
            arguments: {
                length: 0
            },
            scope: undefined,
            f: reference to function f(){}
        },
        Scope: [AO, globalContext.VO],
        this: undefined
    }
```

5.执行f函数，创建f函数执行上下文，f函数执行上下文被压入执行上下文栈

```js
    ECStack = [
        fContext,
        checkscopeContext,
        globalContext
    ];
```

6.f函数执行上下文初始化, 以下跟第4步相同：

1. 复制函数[[scope]]属性创建作用域链
2. 用arguments创建活动对象
3. 初始化活动对象，即加入形参、函数声明、变量声明
4. 将活动对象压入checkscope作用域链顶端

```js
    fContext = {
        AO: {
            arguments: {
                length: 0
            }
        },
        Scope: [AO, checkscopeContext.AO, globalContext.VO],
        this: undefined
    }
```

7.f函数执行，沿着作用域链查找scope值，返回scope值

8.f函数执行完毕，f函数上下文从执行上下文栈中弹出

```js
    ECStack = [
        checkscopeContext,
        globalContext
    ];
```

9.checkscope函数执行完毕，checkscope执行上下文从执行上下文栈中弹出

```js
    ECStack = [
        globalContext
    ];
```

最后关于另一个例子：

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```

大家可以去尝试着模拟它的执行过程。

不过，在下一篇讲闭包的文章中也会提及这个例子的执行过程。

最后，因为都是讲权威指南书上的这个例子，而且写之前看了这篇文章
[https://github.com/kuitos/kuitos.github.io/issues/18](https://github.com/kuitos/kuitos.github.io/issues/18)
写完后总感觉像是抄袭别人的，只能说写的太好，给了我很多影响。感激不尽！

## 相关链接

[《JavaScript深入之执行上下文栈》](https://github.com/mqyqingfeng/Blog/issues/4)

[《JavaScript深入之变量对象》](https://github.com/mqyqingfeng/Blog/issues/5)

[《JavaScript深入之作用域链》](https://github.com/mqyqingfeng/Blog/issues/6)

[《JavaScript深入之从ECMAScript规范解读this》](https://github.com/mqyqingfeng/Blog/issues/7)

## 深入系列

JavaScript深入系列目录地址：[https://github.com/mqyqingfeng/Blog](https://github.com/mqyqingfeng/Blog)。

JavaScript深入系列预计写十五篇左右，旨在帮大家捋顺JavaScript底层知识，重点讲解如原型、作用域、执行上下文、变量对象、this、闭包、按值传递、call、apply、bind、new、继承等难点概念。

如果有错误或者不严谨的地方，请务必给予指正，十分感谢。如果喜欢或者有所启发，欢迎star，对作者也是一种鼓励。