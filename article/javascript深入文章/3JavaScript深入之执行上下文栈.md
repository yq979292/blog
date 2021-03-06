# 执行上下文栈
js代码执行顺序是什么，大部分人都知道同步执行
~~~JS
var foo = function(){
            console.log('foo1');
        }
        foo(); // foo1
        var foo = function(){
            console.log('foo2');
        }
        foo() // foo2
~~~
然后看以下下面代码

~~~js

        function foo(){
            console.log('foo1');0
        }

        foo();// foo2
        function foo(){
            console.log('foo2');
        }
        foo() // foo2

~~~
`js 引擎 执行代码是 一段一段分析执行；不是一行一行执行的` 当一段代码执行完毕后， 会进行一个`准备工作，`比如：第一个例子，`变量提升`，第二个例子中， `函数声明提升`
**准备工作：变量提升，函数提升**

**问题：段 是如何划分的？**
**准备工作是什么时候进行的呢？**

## 可执行代码

这就要说到 JavaScript 的可执行代码(executable code)的类型有哪些了？

其实很简单，就三种，**全局代码、函数代码、eval代码。**

举个例子，当执行到一个函数的时候，就会进行准备工作，这里的“准备工作”，让我们用个更专业一点的说法，就叫做"执行上下文(00context)"。

## 执行上下文栈

接下来问题来了，我们写的函数多了去了，如何管理创建的那么多`执行上下文`呢？

所以 JavaScript 引擎创建了**执行上下文栈**`（Execution context stack，ECS）`来**管理执行上下文**

为了模拟执行上下文栈的行为，让我们定义执行上下文栈是一个**数组**
~~~js
ECStack = [];+
~~~

假设当 JavaScript 开始要解释执行代码的时候，**最先**遇到的就是**全局代码**，所以**初始化**的时候首先就会向**执行上下文栈**添加一个**全局执行上下文**，我们用 globalContext 表示它，并且只有当整个**应用程序结束**的时候，ECStack 才会被**清空**，所以程序结束之前， ECStack 最底部永远有个 globalContext：
~~~js
// 模拟将全局执行上下文添加到执行上下文栈中
ECStack = [
    globalContext
];
~~~
现在 js 引擎 遇到下面的这段代码了：
~~~js
function foo3() {
    console.log('foo3')
}

function foo2() {
    foo3();
}

function foo1() {
    foo2();
}

foo1();
~~~
当**执行一个函数**的时候，就会**创建一个执行上下文**，并且**添加执行上下文栈**，当函数**执行完毕**的时候，就会将函数的执行上下文从**栈中弹出**。知道了这样的工作原理，让我们来看看js引擎如何处理上面这段代码：
~~~

// 模拟js引擎执行代码

// foo1()
ECStack.push(<foo1> functionContext);

// foo1中竟然调用了foo2，还要创建foo2的执行上下文
ECStack.push(<foo2> functionContext);

// foo2还调用了foo3！创建 foo3执行上下文
ECStack.push(<foo3> functionContext);
// ECStack= [globalContext,foo1<functionCountext>,foo2<functionCountext>,foo3<funcrtionCountext>]

// foo3执行完毕 foo3执行上下文销毁
ECStack.pop(); 
// ECStack= [globalContext,foo1<functionCountext>,foo2<functionCountext>]

// foo2执行完毕 foo2执行上下文销毁 
ECStack.pop(); 
// ECStack= [globalContext,foo1<functionCountext>]

// foo1执行完毕，foo1执行上下文销毁
ECStack.pop();
// ECStack= [globalContext]


// javascript接着执行下面的代码，但是ECStack底层永远有个globalContext
// 关闭浏览器 关闭应用程序
// ECStack.pop()
// ECStack = [] 
~~~


## 解答思考题

好啦，现在我们已经了解了执行上下文栈是如何处理执行上下文的，所以让我们看看上篇文章 [《JavaScript深入之词法作用域和动态作用域》](https://www.yuque.com/yanshanzhi/ptxpkh/mbmxid)
最后的问题：

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

两段代码执行的结果一样，但是两段代码究竟有哪些不同呢？

答案就是执行上下文栈的变化不一样。

让我们模拟第一段代码：

```js
ECStack.push(<checkscope> functionContext);
ECStack.push(<f> functionContext);
ECStack.pop();
ECStack.pop();
```

让我们模拟第二段代码：

```js
ECStack.push(<checkscope> functionContext);
ECStack.pop();
ECStack.push(<f> functionContext);
ECStack.pop();
```

是不是有些不同呢？

当然了，这样概括的回答执行上下文栈的变化不同，是不是依然有一种意犹未尽的感觉呢，为了更详细讲解两个函数执行上的区别，我们需要探究一下执行上下文到底包含了哪些内容，所以欢迎阅读下一篇《JavaScript深入之变量对象》。

## 下一篇文章

[《JavaScript深入之变量对象》](https://www.yuque.com/yanshanzhi/ptxpkh/fyh2fx)

## 深入系列

[JavaScript深入系列目录地址](https://www.yuque.com/yanshanzhi/ptxpkh)

JavaScript深入系列预计写十五篇左右，旨在帮大家捋顺JavaScript底层知识，重点讲解如原型、作用域、执行上下文、变量对象、this、闭包、按值传递、call、apply、bind、new、继承等难点概念。

如果有错误或者不严谨的地方，请务必给予指正，十分感谢。如果喜欢或者有所启发，欢迎star，对作者也是一种鼓励。

