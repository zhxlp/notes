# VueAxios 接口请求

## 绑定

- 安装依赖包

  ```bash
  npm install axios vue-axios --save-dev
  ```

- 编辑`main.js`,绑定 axios 到 vue

  ```properties
  import Vue from 'vue'
  import axios from 'axios'
  import VueAxios from 'vue-axios'

  Vue.use(VueAxios, axios)
  ```

- 重启服务,之后就可以使用 `Vue.axios` 或 `this.axios`调用 axios 了。

## axios 基本用法

- Get

  ```javascript
  this.axios
    .get("/user", {
      params: {
        id: 12345,
      },
    })
    .then((response) => {
      console.log(response);
    })
    .catch((error) => {
      console.log(error);
    })
    .finally(() => {});
  ```

- Post

  ```javascript
  this.axios
    .post("/user", {
      firstName: "Fred",
      lastName: "Flintstone",
    })
    .then((response) => {
      console.log(response);
    })
    .catch((error) => {
      console.log(error);
    })
    .finally(() => {});
  ```

## 全局配置

### 请求前缀

- 编辑`main.js`

  ```properties
  axios.defaults.baseURL = '/api';
  ```

### 超时事件

- 编辑`main.js`

  ```properties
  axios.defaults.timeout = 5000;
  ```

## 配置代理

- 编辑`vue.config.js`文件

  ```javascript
  module.exports = {
    devServer: {
      proxy: {
        "/api": {
          target: "https://api.zhxlp.com", // 代理地址
          secure: false, // 忽略ssl验证
          changeOrigin: true, // 是否跨越
          pathRewrite: {
            "/api": "",
          },
        },
      },
    },
  };
  ```

## 拦截

### 请求拦截

- 编辑`main.js`

  ```javascript
  axios.interceptors.request.use(
    function (config) {
      // Do something before request is sent
      return config;
    },
    function (error) {
      // Do something with request error
      return Promise.reject(error);
    }
  );
  ```

### 响应拦截

- 编辑`main.js`

  ```javascript
  axios.interceptors.response.use(
    function (response) {
      // Any status code that lie within the range of 2xx cause this function to trigger
      // Do something with response data
      return response;
    },
    function (error) {
      // Any status codes that falls outside the range of 2xx cause this function to trigger
      // Do something with response error
      return Promise.reject(error);
    }
  );
  ```

## 参考链接

https://www.npmjs.com/package/vue-axios

https://www.npmjs.com/package/axios
