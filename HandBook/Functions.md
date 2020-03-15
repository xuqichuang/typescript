# 函数

## 1. 介绍

和JavaScript一样，TypeScript函数可以创建有名字的函数和匿名函数。 你可以随意选择适合应用程序的方式，不论是定义一系列API函数还是只使用一次的函数。

```typescript
// 具名函数
function add(x, y) {
    return x + y;
}

// 匿名函数
let myAdd = function(x, y) { return x + y; };
```

在JavaScript里，函数可以使用函数体外部的变量。 当函数这么做时，我们说它‘捕获’了这些变量。 至于为什么可以这样做以及其中的利弊超出了本文的范围，但是深刻理解这个机制对学习JavaScript和TypeScript会很有帮助。

```typescript
let z = 100;

function addToZ(x, y) {
    return x + y + z;
}
```

## 2. 函数类型

### 1）为函数定义类型

让我们为上面那个函数添加类型：

```typescript
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```

我们可以给每个参数添加类型之后再为函数本身添加返回值类型。 TypeScript能够根据返回语句自动推断出返回值类型，因此我们通常省略它。

### 2）完整函数类型

现在我们已经为函数指定了类型，下面让我们写出函数的完整类型:

```typescript
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```

函数类型包含两部分：参数类型和返回值类型。 当写出完整函数类型的时候，这两部分都是需要的。 我们以参数列表的形式写出参数类型，为每个参数指定一个名字和类型。 这个名字只是为了增加可读性。 我们也可以这么写：

```typescript
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```

只要参数类型是匹配的，那么就认为它是有效的函数类型，而不在乎参数名是否正确。

返回值类型是函数类型的必要部分，如果函数没有返回任何值，你也必须指定返回值类型为 `void`而不能留空。

### 3）推断类型

尝试这个例子的时候，你会发现如果你在赋值语句的一边指定了类型但是另一边没有类型的话，TypeScript编译器会自动识别出类型：

```typescript
// myAdd has the full function type
let myAdd = function(x: number, y: number): number { return x + y; };

// The parameters `x` and `y` have the type number
let myAdd: (baseValue: number, increment: number) => number =
    function(x, y) { return x + y; };
```

这叫做“按上下文归类”，是类型推论的一种。 它帮助我们更好地为程序指定类型。

## 3. 可选参数和默认参数

TypeScript里的每个函数参数都是必须的。 这不是指不能传递 `null`或`undefined`作为参数，而是说编译器检查用户是否为每个参数都传入了值。 编译器还会假设只有这些参数会被传递进函数。 简短地说，传递给一个函数的参数个数必须与函数期望的参数个数一致。

```typescript
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // ah, just right
```

JavaScript里，每个参数都是可选的，可传可不传。 没传参的时候，它的值就是undefined。 在TypeScript里我们可以在参数名旁使用 `?`实现可选参数的功能。 比如，我们想让last name是可选的：

```typescript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");  // ah, just right
```

可选参数必须跟在必须参数后面。

在TypeScript里，我们也可以为参数提供一个默认值当用户没有传递这个参数或传递的值是`undefined`时。 它们叫做有默认初始化值的参数。 让我们修改上例:

```typescript
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // works correctly now, returns "Bob Smith"
let result2 = buildName("Bob", undefined);       // still works, also returns "Bob Smith"
let result3 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result4 = buildName("Bob", "Adams");         // ah, just right,returns Bob Adams
```

与普通可选参数不同的是，带默认值的参数不需要放在必须参数的后面。 如果带默认值的参数出现在必须参数前面，用户必须明确的传入 `undefined`值来获得默认值。

```typescript
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // okay and returns "Bob Adams"
let result4 = buildName(undefined, "Adams");     // okay and returns "Will Adams"
```

## 4. 剩余参数

必要参数，默认参数和可选参数有个共同点：它们表示某一个参数。 有时，你想同时操作多个参数，或者你并不知道会有多少参数传递进来。 在JavaScript里，你可以使用 `arguments`来访问所有传入的参数。

在TypeScript里，你可以把所有参数收集到一个变量里：

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

这个省略号也会在带有剩余参数的函数类型定义上使用到：

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

## 5. this

学习如何在JavaScript里正确使用`this`就好比一场成年礼。TypeScript能通知你错误地使用了 `this`的地方。

### 1）this和箭头函数

JavaScript里，`this`的值在函数被调用的时候才会指定。

箭头函数能保存函数创建时的 `this`值，而不是调用时的值：

```typescript
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // NOTE: the line below is now an arrow function, allowing us to capture 'this' right here
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

更好事情是，TypeScript会警告你犯了一个错误，如果你给编译器设置了`--noImplicitThis`标记。 它会指出 `this.suits[pickedSuit]`里的`this`的类型为`any`。

### 2）this参数

这是因为 `this`来自对象字面量里的函数表达式。 修改的方法是，提供一个显式的 `this`参数。 `this`参数是个假的参数，它出现在参数列表的最前面：

```typescript
function f(this: void,...args) {
    // make sure `this` is unusable in this standalone function
}
```

让我们往例子里添加一些接口，`Card` 和 `Deck`，让类型重用能够变得清晰简单些：

```typescript
interface Card {
    suit: string;
    card: number;
}
interface Deck {
    suits: string[];
    cards: number[];
    createCardPicker(this: Deck): () => Card;
}
let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // NOTE: The function now explicitly specifies that its callee must be of type Deck
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

现在TypeScript知道`createCardPicker`期望在某个`Deck`对象上调用。 也就是说 `this`是`Deck`类型的，而非`any`，因此`--noImplicitThis`不会报错了。

### 3）this 参数在回调函数中

你可以也看到过在回调函数里的`this`报错，当你将一个函数传递到某个库函数里稍后会被调用时。 因为当回调被调用的时候，它们会被当成一个普通函数调用， `this`将为`undefined`。 稍做改动，你就可以通过 `this`参数来避免错误。 首先，库函数的作者要指定 `this`的类型：

```typescript
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}
```

`this: void` 意味着 `addClickListener` 期望 点击的时候不需要指定 `this` 类型. 其次, 你需要在 `this`这里加上注释：

```typescript
class Handler {
    info: string;
    onClickBad(this: Handler, e: Event) {
        // oops, used this here. using this callback would crash at runtime
        this.info = e.message;
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickBad); // error!
```

指定了`this`类型后，你显式声明`onClickBad`必须在`Handler`的实例上调用。 然后TypeScript会检测到 `addClickListener`要求函数带有`this: void`。所以，你需要 改变 `this`类型来修复这个错误：

```typescript
class Handler {
    info: string;
    onClickGood(this: void, e: Event) {
        // can't use this here because it's of type void!
        console.log('clicked!');
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickGood);
```

因为`onClickGood`指定了`this`类型为`void`，因此传递`addClickListener`是合法的。 当然了，这也意味着不能使用 `this.info`. 如果你两者都想要，你不得不使用箭头函数了：

```typescript
class Handler {
    info: string;
    onClickGood = (e: Event) => { this.info = e.message }
}
```

这是可行的因为箭头函数不会捕获`this`，所以你总是可以把它们传给期望`this: void`的函数。 缺点是每个 `Handler`对象都会创建一个箭头函数。 另一方面，方法只会被创建一次，添加到 `Handler`的原型链上。 它们在不同 `Handler`对象间是共享的。

## 6. 重载

JavaScript本身是个动态语言。 JavaScript里函数根据传入不同的参数而返回不同类型的数据是很常见的。

```typescript
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

`pickCard`方法根据传入参数的不同会返回两种不同的类型。

方法是为同一个函数提供多个函数类型定义来进行函数重载。 编译器会根据这个列表去处理函数的调用。 下面我们来重载 `pickCard`函数。

```typescript
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

这样改变后，重载的`pickCard`函数在调用的时候会进行正确的类型检查。

为了让编译器能够选择正确的检查类型，它与JavaScript里的处理流程相似。 它查找重载列表，尝试使用第一个重载定义。 如果匹配的话就使用这个。 因此，在定义重载的时候，一定要把最精确的定义放在最前面。

注意，`function pickCard(x): any`并不是重载列表的一部分，因此这里只有两个重载：一个是接收对象另一个接收数字。 以其它参数调用 `pickCard`会产生错误。











