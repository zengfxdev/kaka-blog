# 浅谈JS闭包

## 说在前面

要想理解JS的闭包，个人认为要先理解好其他的一些概念：

* 作用域
* JS一切都是按值传递，而不是引用传递。包括函数！

## 问题

```js
function process(data) {
    // 在这里做点有趣的事情
}
var someReallyBigData = {
    // to somting...
};
process(someReallyBigData);
var btn = document.getElementById("my_button");
btn.addEventListener("click", function click(evt) {
    console.log("button clicked");
}, /*capturingPhase=*/ false);
```

`click` 函数的点击回调并不需要 `someReallyBigData` 变量。理论上这意味着当 `process(..)` 执 行后，在内存中占用大量空间的数据结构就可以被垃圾回收了。但是，由于 `click` 函数形成 了一个覆盖整个作用域的闭包，`JavaScript` 引擎极有可能依然保存着这个结构(取决于具体 实现)。


## 延迟垃圾回收机制

```js
function foo() {
    var a = 2;

    function bar() {
        console.log(a);
    }
    return bar;
}
var baz = foo();
baz(); // 2 —— 朋友，这就是闭包的效果。
```

`foo()`执行后，通常情况下`foo()`的整个内部作用域都会被销毁，因为这是引擎的垃圾回收机制。

而神奇的是，`闭包`可以阻止这件事情的发生。事实是`内部作用域`依然存在，因此并没有被引擎回收。

那么谁在使用这个内部作用域呢？答案是：`bar()`函数本身在使用。

拜`bar()`所声明的位置所赐，它拥有涵盖`foo()`内部作用域的闭包，使得该作用域能够一直存活，以供`bar()`在之后任何时间进行引用。

> `bar()`依然持有对该作用域的引用，而这个引用就叫做闭包。


进而，也就解释了文章开头的问题:
`btn`的`click函数`,形成了一个包含整个作用域的闭包,所以当`process(..)`执行完之后，`someReallyBigData`等数据结构并没有被释放！


无论使用何种方式对函数类型的值进行传递，当函数在别处被调用时都可以观察到闭包。

```js
function foo() {
    var a = 2;

    function baz() {
        console.log(a); // 2
    }
    bar(baz);
}

function bar(fn) {
    fn(); // 妈妈快看呀，这就是闭包!
}
```

把内部函数 `baz` 传递给 `bar`，当调用这个内部函数时(现在叫作 `fn`)，它涵盖的 `foo()` 内部
作用域的闭包就可以观察到了，因为它能够访问 `a`。

同样，传递函数也可以是间接的：

```js
var fn;

function foo() {
    var a = 2;

    function baz() {
        console.log(a);
    }
    fn = baz; // 将 baz 分配给全局变量 
}

function bar() {
    fn(); // 妈妈快看呀，这就是闭包!
}
foo();
bar(); // 2
```

无论通过何种手段将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包。


## 闭包作用域

```js
for (var i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000);
}
```

输出5个6

通过iife立即执行函数来创建作用域：

```js
for (var i = 1; i <= 5; i++) {
    (function () {
        setTimeout(function timer() {
            console.log(i);
        }, i * 1000);
    })();
}
```

结果还是输出5个6，
因为`iife`的内部作用域是空的（除了`setTimeout`函数外）,它需要包含一点实质内容才能为我们所用！
如下：

```js
for (var i = 1; i <= 5; i++) {
    (function () {
        var j = i;
        setTimeout(function timer() {
            console.log(j);
        }, j * 1000);
    })();
}
```

这样就正常了！

改进下：

```js
for (var i = 1; i <= 5; i++) {
    (function (j) {
        setTimeout(function timer() {
            console.log(j);
        }, j * 1000);
    })(i);
}
```

在迭代内使用IIFE会为每个迭代都生成一个新的作用域，使得延迟函数的回调可以将新的作用域封闭在每个迭代内部，并都会含有一个具有正确值的变量供我们访问。

上面使用的是在循环内部，使用函数作用域，那么`es6`推出的块作用域呢？

```js
for (var i = 1; i <= 5; i++) {
    let j = i; // 是的，闭包的块作用域! 
    setTimeout(function timer() {
        console.log(j);
    }, j * 1000);
}
```

显示正常!

结合之前学习的作用域了解到let在循环中的用法，加以改进：

```js
for (let i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000);
}
```

显示正常!

可以说很牛逼了：块作用域+闭包的结合!
突然发现整个世界干净了... ...

## 模块

先看看👇的代码：

```js
function foo() {
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log(something);
    }

    function doAnother() {
        console.log(another.join(" ! "));
    }
}
```

这里并没有明显的闭包，只有两个私有数据变量 `something` 和 `another`，以及 `doSomething()` 和 `doAnother()` 两个内部函数，它们的词法作用域(而这 就是闭包)也就是 `foo()` 的内部作用域。

众所周知，函数具有隐藏功能，也就说，外部是无法访问到函数内部定义的变量和函数的！

接着再来看👇的代码：

```js
function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log(something);
    }

    function doAnother() {
        console.log(another.join(" ! "));
    }

    return {
        doSomething,
        doAnother
    };
}

var foo = new CoolModule();     // 不用new也可以
foo.doSomething();      // cool
foo.doAnother();        // 1 ! 2 ! 3
```

🚀🚀这个模式在 `JavaScript` 中被称为模块。最常见的实现模块模式的方法通常被称为模块暴露， 这里展示的是其变体。

1. 但`CoolModule()`只是个函数，必须要通过调用它来创建一个模块实例。如果不执行外部函数，内部作用域和闭包都无法被创建。
	* `var foo = new CoolModule();`

2. `CoolModule()`返回一个用对象字面量语法`{key:value,...}`来表示对象。
	这个返回的对象中含有对内部函数而不是内部数据变量的引用。我们保持内部数据变量是隐藏且私有的状态。
	* 可以将这个对象类型的返回值看作本质上的`模块公共API`.

	```js
	return {
	    doSomething,
	    doAnother
	};
	```

🎉🎉 综上所述，我们通过实践这种方式，已经创造了可以观察和实践闭包的条件了！

模块模式需要具备两个必要条件:

* 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
* 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

上述的例子，当只需要一个实例时，可以稍微改进一下：👇

```js
var foo = (function CoolModule(){
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log(something);
    }

    function doAnother() {
        console.log(another.join(" ! "));
    }

    return {
        doSomething,
        doAnother
    };
})();

foo.doSomething();      // cool
foo.doAnother();        // 1 ! 2 ! 3
```

将模块函数转换成了`IIFE`,并将返回值赋值给`foo`.

## 结尾

`es6`为模块增加了一级语法支持。在通过模块系统进行加载时，`es6`会将文件当作独立的模块来处理。每个模块都可以导入其他模块或特定的API成员，同样也可以导出自己的API成员。

---

这篇文章本质上其实是札记，主要通过阅读<你不知道的JavaScript(上卷)>和其他的一些资料总结的知识。

主要是因为JS的知识点相互之间的关联性很大，如果有某一个知识点不清楚，可能都会影响后续的学习；
比如说这个闭包知识点，必须深刻的理解`作用域`和JS中`值传递`的知识，才能慢慢理解闭包的强大之处!👏🏻👏🏻👏🏻


