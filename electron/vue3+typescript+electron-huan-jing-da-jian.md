# Vue3+TypeScript+Electron 环境搭建

## 环境搭建

- 安装 `vue-cli`

  ```bash
  npm install -g @vue/cli
  ```

- 创建项目

  ```bash
  vue create vue3-electron-demo
  ```

  ```properties
  Vue CLI v4.5.9
  ? Please pick a preset:
    Default ([Vue 2] babel, eslint)
    Default (Vue 3 Preview) ([Vue 3] babel, eslint)
  > Manually select features
  ? Check the features needed for your project:
   (*) Choose Vue version
   (*) Babel
   (*) TypeScript
   ( ) Progressive Web App (PWA) Support
   (*) Router
   (*) Vuex
   (*) CSS Pre-processors
  >(*) Linter / Formatter
   ( ) Unit Testing
   ( ) E2E Testing
  ? Choose a version of Vue.js that you want to start the project with
    2.x
  > 3.x (Preview)
  ? Use class-style component syntax? (y/N) y
  ? Use Babel alongside TypeScript (required for modern mode, auto-detected polyfills, transpiling JSX)? (Y/n) y
  ? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): (Use arrow keys)
  > Sass/SCSS (with dart-sass)
    Sass/SCSS (with node-sass)
    Less
    Stylus
  ? Pick a linter / formatter config:
    ESLint with error prevention only
    ESLint + Airbnb config
    ESLint + Standard config
  > ESLint + Prettier
    TSLint (deprecated)
  ? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)
  >(*) Lint on save
   ( ) Lint and fix on commit
  ? Where do you prefer placing config for Babel, ESLint, etc.? (Use arrow keys)
  > In dedicated config files
    In package.json
  ? Save this as a preset for future projects? (y/N) n
  ```

- 切换工作目录

  ```bash
  cd vue3-electron-demo
  ```

- 配置阿里`electron`源

  ```bash
  npm config set ELECTRON_MIRROR https://npm.taobao.org/mirrors/electron/
  ```

- 集成`electron`

  ```bash
  vue add electron-builder
  ```

  ```properties
  ? Choose Electron Version (Use arrow keys)
    ^7.0.0
    ^8.0.0
  > ^9.0.0
  ```

- 运行

  ```bash
  npm run electron:serve
  ```

- 打包

  ```bash
  npm run electron:build
  ```

## 帮助

https://nklayman.github.io/vue-cli-plugin-electron-builder/guide/guide.html
