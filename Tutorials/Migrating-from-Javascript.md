# JavaScript 迁移

TypeScript不是存在于真空中。 它从JavaScript生态系统和大量现存的JavaScript而来。 将JavaScript代码转换成TypeScript虽乏味却不是难事。 接下来这篇教程将教你怎么做。 在开始转换TypeScript之前，我们假设你已经理解了足够多本手册里的内容。

## 1. 设置目录

如果你在写纯JavaScript，你大概是想直接运行这些`JavaScript`文件， 这些文件存在于 `src`，`lib`或`dist`目录里，它们可以按照预想运行。

若如此，那么你写的纯JavaScript文件将做为TypeScript的输入，你将要运行的是TypeScript的输出。 在从JS到TS的转换过程中，我们会分离输入文件以防TypeScript覆盖它们。 你也可以指定输出目录。

你可能还需要对JavaScript做一些中间处理，比如合并或经过Babel再次编译。 在这种情况下，我们假设你已经设置了这样的目录结构：

```
projectRoot
├── src
│   ├── file1.js
│   └── file2.js
├── built
└── tsconfig.json
```

如果你在`src`目录外还有`tests`文件夹，那么在`src`里可以有一个`tsconfig.json`文件，在`tests`里还可以有一个。

## 2. 编辑配置文件

TypeScript使用`tsconfig.json`文件管理工程配置，例如你想包含哪些文件和进行哪些检查。 让我们先创建一个简单的工程配置文件：

```json
{
    "compilerOptions": {
        "outDir": "./built",
        "allowJs": true,
        "target": "es5"
    },
    "include": [
        "./src/**/*"
    ]
}
```

这里我们为TypeScript设置了一些东西:

1. 读取所有可识别的`src`目录下的文件（通过`include`）。
2. 接受`JavaScript`做为输入（通过`allowJs`）。
3. 生成的所有文件放在`built`目录下（通过`outDir`）。
4. 将`JavaScript`代码降级到低版本比如`ECMAScript 5`（通过`target`）。

现在，如果你在工程根目录下运行`tsc`，就可以在`built`目录下看到生成的文件。 `built`下的文件应该与`src`下的文件相同。 现在你的工程里的TypeScript已经可以工作了。

### 早期收益

现在你已经可以看到TypeScript带来的好处，它能帮助我们理解当前工程。 如果你打开像 [VS Code](https://code.visualstudio.com/)或[Visual Studio](https://visualstudio.com/)这样的编译器，你就能使用像自动补全这样的工具。 你还可以配置如下的选项来帮助查找BUG：

- `noImplicitReturns` 会防止你忘记在函数末尾返回值。
- `noFallthroughCasesInSwitch` 会防止在`switch`代码块里的两个`case`之间忘记添加`break`语句。

TypeScript还能发现那些执行不到的代码和标签，你可以通过设置`allowUnreachableCode`和`allowUnusedLabels`选项来禁用。

## 3. 与构建工具进行集成

在你的构建管道中可能包含多个步骤。 比如为每个文件添加一些内容。 每种工具的使用方法都是不同的，我们会尽可能的包涵主流的工具。

### 1） Gulp

如果你在使用时髦的Gulp，我们已经有一篇关于[使用Gulp](https://www.tslang.cn/docs/handbook/gulp.html)结合TypeScript并与常见构建工具Browserify，Babelify和Uglify进行集成的教程。 请阅读这篇教程。

### 2）Webpack

Webpack集成非常简单。 你可以使用 `ts-loader`，它是一个TypeScript的加载器，结合`source-map-loader`方便调试。 运行：

```bash
$ npm install ts-loader source-map-loader -D
```

并将下面的选项合并到你的`webpack.config.js`文件里：

```js
module.exports = {
    entry: "./src/index.ts",
    output: {
        filename: "./dist/bundle.js",
    },

    // Enable sourcemaps for debugging webpack's output.
    devtool: "source-map",

    resolve: {
        // Add '.ts' and '.tsx' as resolvable extensions.
        extensions: ["", ".webpack.js", ".web.js", ".ts", ".tsx", ".js"]
    },

    module: {
        loaders: [
            // All files with a '.ts' or '.tsx' extension will be handled by 'ts-loader'.
            { test: /\.tsx?$/, loader: "ts-loader" }
        ],

        preLoaders: [
            // All output '.js' files will have any sourcemaps re-processed by 'source-map-loader'.
            { test: /\.js$/, loader: "source-map-loader" }
        ]
    },

    // Other options...
};
```

要注意的是`ts-loader`必须在其它处理`.js`文件的加载器之前运行。 

## 4. 转换到TypeScript文件

到目前为止，你已经做好了使用`TypeScript`文件的准备。 第一步，将 `.js`文件重命名为`.ts`文件。 如果你使用了`JSX`，则重命名为 `.tsx`文件。

如果你不想让TypeScript将没有明确指定的类型默默地推断为 `any`类型，可以在修改文件之前启用`noImplicitAny`。 你可能会觉得这有些过度严格，但是长期收益很快就能显现出来。

在转换后将会看到错误信息。 重要的是我们要逐一的查看它们并决定如何处理。 通常这些都是真正的BUG，但有时必须要告诉TypeScript你要做的是什么。

### 1）由模块导入

你可能会看到一些类似`Cannot find name 'require'.`和`Cannot find name 'define'.`的错误。 遇到这种情况说明你在使用模块。 你仅需要告诉TypeScript它们是存在的：

```typescript
// For Node/CommonJS
declare function require(path: string): any;
```

或者

```typescript
// For RequireJS//AMD
declare function define(...args: any[]): any;
```

最好是避免使用这些调用而改用TypeScript的导入语法

首先，你要使用TypeScript的`module`标记来启用一些模块系统，可用的选项有`commonjs`, `amd`, `system`,`umd`。

例如：

```js
var foo = require('foo')
foo.doStuff();
```

用TS语法改写

```js
import foo = require('foo')
foo.doSuff()
```

### 2）获取声明文件

js 转换成 ts 可能会遇到 `cannot find module 'foo'` 这样的错误，问题出在没有声明文件来描述你的代码库，幸运的是这非常简单，如果TS抱怨像是没有`lodash`包，那你只需要安装声明文件就可以

```js
npm install -s @types/lodash
```

如果你没有使用 `commonjs`模块选项，那么就需要讲`moduleResolution`选项设置为`node`

之后，你就可以导入`lodash`了，并且会获得精确的自动补全功能。

### 3）由模块导出

通常来说，模块导出涉及到`exports` 或者 `module.exports`。TS 允许你是用顶级的导出语句，比如：

导出匿名函数

```js
module.exports.feedPets = function(pets){
	//...
}
```

TS改写：

```js
export function(pets){
	// ...
}
```

导出函数名:

```js
function foo(){
 // ...
}
```

TS 改写：

```js
function foo(){
	// ...
}

export = foo
```

### 4）过多或过少的参数

有时你会发现你在调用一个具有过多或者过少参数的函数。通常，这是一个BUG，单在某些情况下，你可以声明一个使用`arguments`对象而不需要写出所有的参数

```js
function myCoolFunction() {
    if (arguments.length == 2 && !Array.isArray(arguments[1])) {
        var f = arguments[0];
        var arr = arguments[1];
        // ...
    }
    // ...
}

myCoolFunction(function(x) { console.log(x) }, [1, 2, 3, 4]);
myCoolFunction(function(x) { console.log(x) }, 1, 2, 3, 4);
```

这种情况下，我们需要利用TypeScript的函数重载来告诉调用者 `myCoolFunction`函数的调用方式。

```js
function myCoolFunction(f: (x: number)=> void, nums: number[]): void;
function myCoolFunction(f: (x: number)=> void, ...nums: number[]): void;

function myCoolFunction() {
    if(arguments.length == 2 && !Array.isArray(arguments[1])){
        var f = arguments[0]
        var arr = arguments[1]
        // ...
    }
    // ...
}
```

我们为 `myCoolFunction` 函数添加了两个重载签名。第一个检查`myCoolFunction`函数是否接受一个函数（它又接受一个`number`参数）和一个`number`数组。第二个同样接收了一个函数，并且剩余参数（`...nums`）来表示之后的其他所有参数必须是`number`类型

### 5）连续添加属性

有些人可能因为代码美观性而喜欢先创建一个对象然后立即添加属性

```js
var option = {}
option.color = 'red';
option.volum = 11
```

TS会提示你不能给`color`和`volum`赋值，因为先前指定`option`的类型为`{}`并不带有任何属性。如果你将声明变成对象字面量将不会产生错误。

```js
var option = {
	color: 'red',
	volum : 11
}
```

你还可以定义`option`的类型并且添加类型断言到对象字面量上

```js
interface Option {color: string, volum: number}

let option = {} as Option
option.color = 'red'
option.volum = 11
```

你也可以把`option`指定成`any`类型，这是最简单的，也是受益最少的

`any`,`Object`,`{}`区别

`any`是最普通的类型但是允许你在上面做任何事情，也就是你可以在上面调用，构造，访问他的属性等等。但是当你使用`any`时，也就意味着你将失去大多数 TS 听歌的错误检查和编译器支持。

如果你还是决定使用`Object`和`{}`，虽说他们基本一样，但是从技术层面说，`{}`在一些深奥的情况里比`Object`更普通

### 6）启用严格检查

TypeScript提供了一些检查来保证安全以及帮助分析你的程序。 当你将代码转换为了TypeScript后，你可以启用这些检查来帮助你获得高度安全性。

#### （1）没有隐式的any

在某些情况下TypeScript没法确定某些值的类型。 那么TypeScript会使用 `any`类型代替。 这对代码转换来讲是不错，但是使用 `any`意味着失去了类型安全保障，并且你得不到工具的支持。 你可以使用 `noImplicitAny`选项，让TypeScript标记出发生这种情况的地方，并给出一个错误。

#### （2）严格的 null 和 undefined 替换

默认地，TypeScript把`null`和`undefined`当做属于任何类型。 这就是说，声明为 `number`类型的值可以为`null`和`undefined`。 因为在JavaScript和TypeScript里， `null`和`undefined`经常会导致BUG的产生，所以TypeScript包含了`strictNullChecks`选项来帮助我们减少对这种情况的担忧。

当启用了`strictNullChecks`，`null`和`undefined`获得了它们自己各自的类型`null`和`undefined`。 当任何值 *可能*为`null`，你可以使用联合类型。 比如，某值可能为 `number`或`null`，你可以声明它的类型为`number | null`。

假设有一个值TypeScript认为可以为`null`或`undefined`，但是你更清楚它的类型，你可以使用`!`后缀。

```typescript
declare var foo: string[] | null;
foo.length; // error - 'foo' is possibly 'null'
foo!.length; // okay - 'foo!' just has type 'string[]'
```

要当心，当你使用`strictNullChecks`，你的依赖也需要相应地启用`strictNullChecks`。

#### （3）this 没有隐式的 any 

当你在类的外部使用`this`关键字时，它会默认获得`any`类型。 比如，假设有一个 `Point`类，并且我们要添加一个函数做为它的方法：

```typescript
class Point {
	constructor(public x, public y){
		
	}
	getDistance(p: Point){
		let dx = p.x - this.x;
		let dy = p.y - this.y;
		return Msth.sqrt(dx**2 + dy**2)
	}
}

// ...

// 重开断言
interface Point {
	distanceFromOrigin(point: Point): number;
}
Point.prototype.distanceFromOrigin = function(point: Point){
	return this.getDistance({x: 0, y: 0})
}
```

这就产生了我们上面提到的错误 - 如果我们错误地拼写了`getDistance`并不会得到一个错误。 正因此，TypeScript有 `noImplicitThis`选项。 当设置了它，TypeScript会产生一个错误当没有明确指定类型（或通过类型推断）的 `this`被使用时。 解决的方法是在接口或函数上使用指定了类型的 `this`参数：

下面是原型链改写

```typescript
Point.prototype.distanceFromOrigin = function(this: Point, point: Point){
	return this.getDistance({x: 0, y: 0});
}
```

