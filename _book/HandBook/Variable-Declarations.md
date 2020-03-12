# 变量声明

## 1. var

一直以来我们都是通过`var`关键字定义JavaScript变量：

```typescript
var num = 10;
```

我们也可以在函数内部定义变量：

```typescript
function f() {
    var message = "Hello, world!";

    return message;
}
```

并且我们也可以在其它函数内部访问相同的变量。

```typescript
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // returns 11;
```

上面的例子里，`g`可以获取到`f`函数里定义的`a`变量。 每当 `g`被调用时，它都可以访问到`f`里的`a`变量。 即使当 `g`在`f`已经执行完后才被调用，它仍然可以访问及修改`a`。

```typescript
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns 2
```

### 作用域规则

对于熟悉其它语言的人来说，`var`声明有些奇怪的作用域规则。 看下面的例子：

```typescript
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```

变量 `x`是定义在*`if`语句里面*，但是我们却可以在语句的外面访问它。 这是因为 `var`声明可以在包含它的函数，模块，命名空间或全局作用域内部任何位置被访问（我们后面会详细介绍），包含它的代码块对此没有什么影响。 有些人称此为 `var`作用域或函数作用域。 函数参数也使用函数作用域。

这些作用域规则可能会引发一些错误。 其中之一就是，多次声明同一个变量并不会报错：

```typescript
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

这里很容易看出一些问题，里层的`for`循环会覆盖变量`i`，因为所有`i`都引用相同的函数作用域内的变量。 有经验的开发者们很清楚，这些问题可能在代码审查时漏掉，引发无穷的麻烦。

### 捕获变量怪异之处

快速的猜一下下面的代码会返回什么：

```typescript
for (var i = 0; i < 5; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

介绍一下，`setTimeout`会在若干毫秒的延时后执行一个函数（等待其它代码执行完毕）。

运行结果：

```typescript
10
10
10
10
10
```

很多JavaScript程序员对这种行为已经很熟悉了，但如果你很不解，你并不是一个人。 大多数人期望输出结果是这样：

```typescript
0
1
2
3
4
```

> 我们传给`setTimeout`的每一个函数表达式实际上都引用了相同作用域里的同一个`i`。

`setTimeout`在若干毫秒后执行一个函数，并且是在`for`循环结束后。 `for`循环结束后，`i`的值为`5`。 所以当函数被调用的时候，它会打印出 `5`！

一个通常的解决方法是使用立即执行的函数表达式（IIFE）来捕获每次迭代时`i`的值：

```typescript
for (var i = 0; i < 5; i++) {
    // 捕获'i'的当前状态
    // 通过调用具有其当前值的函数
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```

参数 `i`会覆盖`for`循环里的`i`，但是因为我们起了同样的名字，所以我们不用怎么改`for`循环体里的代码。

## 2. let 

`let`与`var`的写法一致。

```typescript
let hello = "Hello!";
```

### 块作用域

当用`let`声明一个变量，它使用的是*词法作用域*或*块作用域*。

块作用域变量在包含它们的块或`for`循环之外是不能访问的。

```typescript
function f(input: boolean) {
    let a = 100;

    if (input) {
        // Still okay to reference 'a'
        let b = a + 1;
        return b;
    }

    // Error: 'b' doesn't exist here
    return b;
}
```

这里我们定义了2个变量`a`和`b`。 `a`的作用域是`f`函数体内，而`b`的作用域是`if`语句块里。

在`catch`语句里声明的变量也具有同样的作用域规则。

```typescript
try {
    throw "oh no!";
}
catch (e) {
    console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```

拥有块级作用域的变量的另一个特点是，它们不能在被声明之前读或写。 虽然这些变量始终“存在”于它们的作用域里，但在直到声明它的代码之前的区域都属于 *暂时性死区*。它只是用来说明我们不能在 `let`语句之前访问它们，幸运的是TypeScript可以告诉我们这些信息。

```typescript
a++; // illegal to use 'a' before it's declared;
let a;
```

### 重定义和屏蔽

我们提过使用`var`声明时，它不在乎你声明多少次；你只会得到1个。

```typescript
function f(x) {
    var x;
    var x;

    if (true) {
        var x;
    }
}
```

`let`声明就不会这么宽松:

```typescript
let x = 10;
let x = 20; // 错误，不能在1个作用域里多次声明`x`
```

同样`let`声明和`var`声明，以及参数名重复时，也会给出错误

```typescript
function f(x) {
    let x = 100; // error: interferes with parameter declaration
}

function g() {
    let x = 100;
    var x = 100; // error: can't have both declarations of 'x'
}
```

在一个嵌套作用域里引入一个新名字的行为称做*屏蔽*。 它是一把双刃剑，它可能会不小心地引入新问题，同时也可能会解决一些错误。 例如，假设我们现在用 `let`重写之前的`sumMatrix`函数。

```typescript
function sumMatrix(matrix: number[][]) {
    let sum = 0;
    for (let i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (let i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

这个版本的循环能得到正确的结果，因为内层循环的`i`可以屏蔽掉外层循环的`i`。

*通常*来讲应该避免使用屏蔽，因为我们需要写出清晰的代码。 同时也有些场景适合利用它。

### 块级作用域变量的获取

用`var`声明的变量时，就算作用域内代码已经执行完毕，这个环境与其捕获的变量依然存在。

```typescript
function theCityThatAlwaysSleeps() {
    let getCity;

    if (true) {
        let city = "Seattle";
        getCity = function() {
            return city;
        }
    }

    return getCity();
}
```

因为我们已经在`city`的环境里获取到了`city`，所以就算`if`语句执行结束后我们仍然可以访问它。

当`let`声明出现在循环体里时拥有完全不同的行为。 不仅是在循环里引入了一个新的变量环境，而是针对 *每次迭代*都会创建这样一个新作用域。 这就是我们在使用立即执行的函数表达式时做的事，所以在 `setTimeout`例子里我们仅使用`let`声明就可以了。

```typescript
for (let i = 0; i < 5 ; i++) {
    setTimeout(function() {console.log(i); }, 100 * i);
}
```

会输出与预料一致的结果：

```typescript
0
1
2
3
4
```

## 3. const

`const` 声明是声明变量的另一种方式。

```typescript
const numLivesForCat = 9;
```

它们与`let`声明相似，但是就像它的名字所表达的，它们被赋值后不能再改变。 换句话说，它们拥有与 `let`相同的作用域规则，但是不能对它们重新赋值。

这很好理解，它们引用的值是*不可变的*。

```typescript
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

除非你使用特殊的方法去避免，实际上`const`变量的内部状态是可修改的。

## 4. let VS const

使用最小特权原则，所有变量除了你计划去修改的都应该使用`const`，使用 `const`也可以让我们更容易的推测数据的流动。

## 5. 解构

### 解构数组

最简单的解构莫过于数组的解构赋值了：

```typescript
let input = [1, 2];
let [first, second] = input;
console.log(first); // 1
console.log(second); // 2
```

解构作用于已声明的变量会更好：

```
// 交换变量
[first, second] = [second, first];
```

作用于函数参数：

```typescript
function f([first, second]: [number, number]) {
    console.log(first);
    console.log(second);
}
f(input);
```

你可以在数组里使用`...`语法创建剩余变量：

```typescript
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // 1
console.log(rest); // [ 2, 3, 4 ]
```

当然，由于是JavaScript, 你可以忽略你不关心的尾随元素：

```typescript
let [first] = [1, 2, 3, 4];
console.log(first); // 1
```

或其它元素：

```typescript
let [, second, , fourth] = [1, 2, 3, 4];
```

### 结构对象

```typescript
let o = {
    a: "foo",
    b: 12,
    c: "bar"
};
let { a, b } = o;
```

这通过 `o.a` and `o.b` 创建了 `a` 和 `b` 。 注意，如果你不需要 `c` 你可以忽略它。

就像数组解构，你可以用没有声明的赋值：

```typescript
({ a, b } = { a: "baz", b: 101 });
a // "baz"
b // 101
```

注意，我们需要用括号将它括起来，因为Javascript通常会将以 `{` 起始的语句解析为一个块。

你可以在对象里使用`...`语法创建剩余变量：

```typescript
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;
total // 15
```

### 属性重命名

你也可以给属性以不同的名字：

```typescript
let { a: newName1, b: newName2 } = o;
```

令人困惑的是，这里的冒号*不是*指示类型的。 如果你想指定它的类型， 仍然需要在其后写上完整的模式。

```typescript
let {a, b}: {a: string, b: number} = o;
```

### 默认值

默认值可以让你在属性为 undefined 时使用缺省值：

```typescript
function keepWholeObject(wholeObject: { a: string, b?: number }) {
    let { a, b = 1001 } = wholeObject;
}
```

现在，即使 `b` 为 undefined ， `keepWholeObject` 函数的变量 `wholeObject` 的属性 `a` 和 `b` 都会有值。

### 函数声明

解构也能用于函数声明。 看以下简单的情况：

```typescript
type C = { a: string, b?: number }
function f({ a, b }: C): void {
    // ...
}
```

但是，通常情况下更多的是指定默认值，解构默认值有些棘手。 首先，你需要在默认值之前设置其格式。

```typescript
function f({ a="", b=0 } = {}): void {
    // ...
}
f();
```

其次，你需要知道在解构属性上给予一个默认或可选的属性用来替换主初始化列表。 要知道 `C` 的定义有一个 `b` 可选属性：

```typescript
function f({ a, b = 0 } = { a: "" }): void {
    // ...
}
f({ a: "yes" }); // ok, default b = 0
f(); // ok, default to {a: ""}, which then defaults b = 0
f({}); // error, 参数'a' 是必须赋值的 
```

### 展开

展开操作符正与解构相反。 它允许你将一个数组展开为另一个数组，或将一个对象展开为另一个对象。 例如：

```typescript
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
console.log(bothPlus) // [0, 1, 2, 3, 4, 5]
```

展开操作创建了 `first`和`second`的一份浅拷贝。 它们不会被展开操作所改变。

还可以展开对象：

```typescript
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };
console.log(search) // { food: "rich", price: "$$", ambiance: "noisy" }
```

像数组展开一样，它是从左至右进行处理，但结果仍为对象。 这就意味着出现在展开对象后面的属性会覆盖前面的属性。 因此，如果我们修改上面的例子，在结尾处进行展开的话：

```typescript
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { food: "rich", ...defaults };
console.log(search) // { food: "spicy", price: "$$", ambiance: "noisy" }
```

对象展开还有其它一些意想不到的限制。 首先，它仅包含对象自身的可枚举属性。 大体上是说当你展开一个对象实例时，你会丢失其方法：

```typescript
class C {
  p = 12;
  m() {
  }
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```

TypeScript编译器不允许展开泛型函数上的类型参数。







