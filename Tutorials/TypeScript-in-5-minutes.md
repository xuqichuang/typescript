# 5分钟上手Typescript

让我们使用TypeScript来创建一个简单的Web应用。

## 1. 安装

```js
npm install -g typescript
```

## 2. 构建你的第一个TypeScript文件

在编辑器，将下面的代码输入到`greeter.ts`文件

```js
function greeter(person) {
    return "Hello, " + person;
}

let user = "Jane User";

console.log(greeter(user))
```

然后在当前目录下运行 `ts`文件

```
tsc greeter.ts
```

执行命令之后会创建一个转义成`ES5`的`greeter.js`文件，它包含了和输入文件中相同的JavsScript代码。

接下来让我们看看TypeScript工具带来的高级功能。 给 `person`函数的参数添加`: string`类型注解，如下：

```js
function greeter(person: string) {
    return "Hello, " + person;
}

let user = "Jane User";

console.log(greeter(user))
```

## 3. 类型注解

TypeScript里的类型注解是一种轻量级的为函数或变量添加约束的方式。 在这个例子里，我们希望 `greeter`函数接收一个字符串参数。 然后尝试把 `greeter`的调用改成传入一个数组：

```js
function greeter(person: string) {
    return "Hello, " + person;
}

let user = [0, 1, 2];
console.log(greeter(user))
```

重新编译，你会看到产生了一个错误。

```
greeter.ts(6,21): error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.
```

类似地，尝试删除`greeter`调用的所有参数。 TypeScript会告诉你使用了非期望个数的参数调用了这个函数。 在这两种情况中，TypeScript提供了静态的代码分析，它可以分析代码结构和提供的类型注解。

要注意的是尽管有错误，`greeter.js`文件还是被创建了。 就算你的代码里有错误，你仍然可以使用TypeScript。但在这种情况下，TypeScript会警告你代码可能不会按预期执行。

## 4. 接口

让我们开发这个示例应用。这里我们使用接口来描述一个拥有`firstName`和`lastName`字段的对象。 在TypeScript里，只在两个类型内部的结构兼容那么这两个类型就是兼容的。 这就允许我们在实现接口时候只要保证包含了接口要求的结构就可以，而不必明确地使用 `implements`语句。

```js
interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person: Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = { firstName: "Jane", lastName: "User" };
console.log(greeter(user))
```

## 5. 类

最后，让我们使用类来改写这个例子。 TypeScript支持JavaScript的新特性，比如支持基于类的面向对象编程。

让我们创建一个`Student`类，它带有一个构造函数和一些公共字段。 注意类和接口可以一起共作，程序员可以自行决定抽象的级别。

还要注意的是，在构造函数的参数上使用`public`等同于创建了同名的成员变量。

```js
class Student {
    fullName: string;
    constructor(public firstName, public middleInitial, public lastName) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
}

interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person : Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", "User");
console.log(greeter(user))
```

重新运行`tsc greeter.ts`，你会看到生成的JavaScript代码和原先的一样。 TypeScript里的类只是JavaScript里常用的基于原型面向对象编程的简写。