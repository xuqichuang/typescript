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

第一步达成？ 太棒了! 你已经成功地将一个文件从`JavaScript`转换成了`TypeScript`!

