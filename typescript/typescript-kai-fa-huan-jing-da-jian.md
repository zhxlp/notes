# TypeScript 开发环境搭建

## 初始化项目

- 创建项目目录

  创建目录`demo`，打开终端，切换到`demo`目录

- 初始化 npm

  ```bash
  npm init -y
  ```

- 安装依赖

  ```bash
  npm install --save-dev typescript @types/node
  ```

- 初始化 TypeScript 配置

  ```bash
  node_modules\.bin\tsc --init
  ```

- 创建源码目录和打包目录

  在`demo`目录创建`src`和`dist`目录，目录结构如下

  ```bash
  demo
  ├─dist
  └─src
  └─package.json
  └─tsconfig.json
  ```

- 配置`tsconfig.json`

  ```properties
  "outDir": "./dist",
  "root": "./src",
  ```

- 创建代码文件

  创建`./src/index.ts`文件

  ```typescript
  const hello: string = "Hello World";
  console.log(hello);
  ```

- 编译并运行

  ```bash
  # 编译
  tsc
  # 运行
  node ./dist/index.js
  ```

## 代码检查

- 安装依赖

  ```bash
  npm install eslint typescript @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-config-airbnb-typescript --save-dev
  ```

- 初始化 eslinit 配置文件

  ```bash
  node_modules\.bin\eslint --init
  ```

  ```properties
  ? How would you like to use ESLint? ...  // 您想怎样使用ESLint
    To check syntax only  // 只检查语法
    To check syntax and find problems  // 检查语法并发现问题
  > To check syntax, find problems, and enforce code style  // 检查语法、发现问题并执行代码样式
  ? What type of modules does your project use? ...  // 您的项目使用什么类型的模块?
  > JavaScript modules (import/export)
    CommonJS (require/exports)
    None of these  // 这些都不是
  ? Which framework does your project use? ...  // 项目中使用的什么框架？
    React
    Vue.js
  > None of these
  ? Does your project use TypeScript?  // 你的项目使用TypeScript吗？
    No
  > Yes
  ? Where does your code run? ... (Press <space> to select, <a> to toggle all, <i> to invert selection)
  // 你的代码在哪里运行？（按<space>选择，<a>切换全部，<i>反转选择）
    Browser
  √ Node
  ? How would you like to define a style for your project? ...  // 您希望如何定义项目的样式？
  > Use a popular style guide  // 使用流行的风格指南
    Answer questions about your style // 回答关于你的风格的问题
    Inspect your JavaScript file(s) // 检查JavaScript文件
  ? Which style guide do you want to follow? ...  // 您想遵循哪种风格指南?
  > Airbnb: https://github.com/airbnb/javascript
    Standard: https://github.com/standard/standard
    Google: https://github.com/google/eslint-config-google
  ? What format do you want your config file to be in? ...  // 您希望配置文件的格式是什么?
  > JavaScript
    YAML
    JSON
  ? Would you like to install them now with npm? ... // 你想现在用npm安装吗？
    No
  > Yes
  ```

- 修改`.eslintrc.js`

  ```javascript
  module.exports = {
    env: {
      es6: true,
      node: true,
    },
    extends: [
      "airbnb-base",
      "airbnb-typescript/base",
      "eslint:recommended",
      "plugin:@typescript-eslint/recommended",
      "plugin:@typescript-eslint/recommended-requiring-type-checking",
    ],
    parser: "@typescript-eslint/parser",
    parserOptions: {
      sourceType: "module",
      project: ["./tsconfig.json"],
      createDefaultProgram: true,
    },
    plugins: ["@typescript-eslint"],
    rules: {
      "no-console": "off",
      "no-debugger": "off",
    },
  };
  ```

- 创建`.eslintignore`文件

  ```properties
  node_modules/
  dist/
  ```

### VSCode 集成 ESLint 检查

- 安装 ESLint 插件

  点击「扩展」按钮，搜索 ESLint

- 修改 VSCode 配置

  VSCode 中的 ESLint 插件默认是不会检查 .ts 后缀的，需要在「文件 => 首选项 => 设置」中，添加以下配置：

  ```properties
  "eslint.validate": [
  ...
  "typescript"
  ]
  ```

## 调试

- 安装依赖

  ```bash
  npm install ts-node --save-dev
  ```

- 在 `VScode` 调试中，添加配置

  ```properties
  {
      // 使用 IntelliSense 了解相关属性。
      // 悬停以查看现有属性的描述。
      // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
      "version": "0.2.0",
      "configurations": [

          {
              "type": "node",
              "request": "launch",
              "name": "Launch Project",
              "runtimeArgs": [
                  "-r",
                  "ts-node/register"
              ],
              "args": [
                  "${workspaceFolder}/src/index.ts"
              ],
              "sourceMaps": true,
              "cwd": "${workspaceRoot}",
              "protocol": "inspector",
              "console": "integratedTerminal",
              "internalConsoleOptions": "neverOpen"
          },
          {
              "name": "Current TS File",
              "type": "node",
              "request": "launch",
              "args": [
                "${relativeFile}"
              ],
              "runtimeArgs": [
                "-r",
                "ts-node/register"
              ],
              "sourceMaps": true,
              "cwd": "${workspaceRoot}",
              "protocol": "inspector",
              "console": "integratedTerminal",
              "internalConsoleOptions": "neverOpen"
            }
      ]
  }
  ```

## 路径别名

- 修改`tsconfig.json`如下字段

  ```properties
  "baseUrl": "./",
  "paths": {
    "@/*":["./src/*"]
  },
  ```

### ts-node 和 mocha 报错

在默认状态下，ts-node 不会受 tsconfig 中别名配置的影响，想要让 ts-node 配置别名，需要安装`tsconfig-paths`

```properties
npm install tsconfig-paths --save-dev
```

运行命令

```bash
# ts-node使用
ts-node -r tsconfig-paths/register main.ts
#mocha配合ts-node使用
mocha -r ts-node/register -r tsconfig-paths/register "test/**/*.ts"
```

### ESLint 报错

- 安装依赖

  ```bash
  npm install eslint-import-resolver-alias --save-dev
  ```

- 配置`.eslintrc.js`

  ```properties
    settings: {
      'import/resolver': {
        alias: {
          map: [
            ['@', './src'],
          ],
          extensions: ['.ts', '.js', '.jsx', '.json'],
        },
      },
    },
  ```

### node 运行报错

- 安装依赖

  ```bash
  npm install module-alias --save
  npm install @types/module-alias --save-dev
  ```

- 在程序主文件头部添加路径别名

  ```javascript
  import moduleAlias from "module-alias";
  moduleAlias.addAliases({
    "@": __dirname,
  });
  ```

## 参考

https://segmentfault.com/a/1190000018777683

https://zhuanlan.zhihu.com/p/298189197
