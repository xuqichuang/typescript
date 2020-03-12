# 基础类型

## 1. 布尔值

`boolean`，可以定义为`true`/`false`

```typescript
let isDone: boolean = false
```

## 2. 数值

`number`，所有的数字都是浮点数，支持十进制，十六进制，二进制和八进制

```typescript
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
let binaryLiteral: number = 0b1010;
let octalLiteral: number = 0o744;
```

## 3. 字符串

`string`，可以使用双引号（`""`）、单引号（`''`）和反引号（` `` `）表示

```typescript
let name: string = 'bob';
let sex: string = "male";
let sentence: string = `hello, my name is ${name}`;
```

## 4. 数组

有两种方法可以定义数组

1. 在元素类型后面接上`[]`，表示由此类型元素组成一个数组

```typescript
let list: number[] = [1, 2, 3];
```

2. 使用数组泛型，`Array<元素类型>`

```typescript
let list: Array<number> = [1, 2, 3];
```

## 5. 元组 Tuple

元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同，比如你可以定义一对值分别为`string`和`number`类型的元组

```typescript
let x: [string, number];
x = ['hello', 10]; // ok
x = [10, 'hello']; // error
```

当访问一个一直索引的元素，会得到正确的类型

```typescript
console.log(x[0].substr(1)); // ello
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

当访问一个跨界的元素，会使用联合类型替代

```typescript
x[3] = 'world';// OK, 字符串可以赋值给(string | number)类型
console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString
x[6] = true; // Error, 布尔不是(string | number)类型
```

## 6. 枚举

`enum`类型是对JavaScript标准数据类型的一个补充

```typescript
enum Color {Red, Green, Yellow}
let c: Color = Color.Green
```

默认情况下，从0开始为元素编号，你也可以手动指定成员的数值。例如，我们将上面的例子改成从`1`开始编号：

```typescript
enum Color {Red=1, Green, Yellow}
let c: Color = Color.Green
```

也可以全部手动赋值

```typescript
enum Color {Red=1, Green=2, Yellow=4}
let colorName: string = Color[2]
console.log(colorName) // 显示'Green'因为上面代码里它的值是2
```

## 7. Any

`any`，有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。这些值可能来子推燕动态的内容，比如用户输入或者第三方代码库。这种情况下，我们不希望类型检查器对这些值进行检查而是直接让他们通过编译阶段的检查，那我们就可使用`any`类型来标记这些变量：

```typescript
let notSure: any = 4;
notSure = 'hello';
notSure = true;
```

在对现有代码进行改写的时候，`any`类型是十分有用的。他允许你在变异时可选择地包含或移除类型检查。你可能认为`Object`有相似的作用，就想他在其他语言中的那样。但是`Object`类型的变量只是允许你给他赋任意值，但是却不能够在他上面调用任何方法，几遍他真的有这些方法：

```typescript
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'
```

当你只知道一部分数据类型时，`any`类型也是有用的。比如，你有一个数组，他包含了不同的类型的数据：

```typescript
let list: any[] = [1, 'hello', true];

list[1] = 100;
```

## 8. Void

`void`，某种程度上来说`void`类型和`any`类型相反，他标书没有任何类型。当一个函数没有返回之时，你通常会见到其返回值类型为`void`：

```typescript
function warnUser(): void{
	console.log('This is my warning message');
}
```

声明一个`void`类型的变量没有什么大用，因为你指能赋予它`undefined`和`null`

```typescript
let unusable: void = undefined;
```

## 9. Null 和 Undefined

`null`的类型就是`null`

`undefined`的类型就是`undefined`

和`void`相似，他们本身的类型用处不是很大：

```typescript
// 我们不能给这些变量赋太多的值
let u: undefined = undefined;
let n: null = null;
```

默认情况下，`null`和`undefined`是所有类型的子类型，也就是说你可以把`null`和`undefined`赋值给`number`类型的变量

然而，当你制定了`--strictNullChecks`标记时，`null`和`undefined`只能赋值给`void`和他们各自。这能避免很多常见的问题，允许在某处你想传入一个`string`或`null`或`undefined`，你可以使用联合类型`string | null | undefined`。

> 注意： 我们应该尽可能的使用 `--strictNullChecks`

## 10. Never

`never`类型表示哪些用不存在值得类型，例如：`never`类型是那些总会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型；变量也可能是`never`类型，当他们被用不为真的类型保护所约束时。

`never`类型是任意类型的子类型，也可以赋值给任意类型；然而，没有类型是`never`的子类型或可以赋值给`never`类型，（除了`never`本身之外）。即使`any`也不可以赋值给`never`。

下面是一些返回`never`类型的函数：

```typescript
// 返回 never类型的函数必须存在无法达到的终点，抛出异常
function error(message: string): never{
	throw new Error(message);
}

// 推断类型的返回值类型为never, 返回一个错误
function fail(){
    return error('something failed');
}

// 返回never函数必须存在无法到达的终点
function infiniteLoop(): never {
    while (true){
         // ...  
    }
}
```

## 11. Object

`object`表示非原始类型，也就是除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的类型。使用`object`类型，就可以更好的表示像`Object.create`这样的 API，例如：

```typescript
declare function create(o: object | null): void;

create({prop: 0}); // ok
create(null); // ok

create(42); // Error
create('string'); // Error
create(false); // Error
create(undefined); // Error
```

## 12. 类型断言

有些时候，你会比TypeScript更了解某个值的详细信息，通常这会发生在你清楚地知道一个实体具有比他现有类型更确切的类型。

通过类型断言这种方式告诉编译器，“相信我，我知道自己在干什么”。类型断言好比其他语言中的类型转换，但是不进行特殊的数据检查和结构。他没有运行时的影响，只在编译阶段起作用， TypeScript 会假设你已经进行了必须的检查。

类型断言有两种形式：

1. “尖括号”语法：

```typescript
let someValue: any = 'this is a string';
let strLength: number = (<string>someValue).length;
```

2. `as`语法

```typescript
let someValue: any = 'this is a string';
let strLength: number = (someValue as string).length
```

两种形式是等价的。至于使用哪种写法全凭个人喜好；然而，当你在TypeScript里使用JSX时，只有`as`语法断言是被允许的。







































