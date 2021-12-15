- [从 0 开始手把手带你搭建一套规范的 Vue3.x 项目工程环境](https://juejin.cn/post/6951649464637636622)

作者：XPoet    来源：掘金
链接：https://juejin.cn/post/6951649464637636622



## 技术栈

- 编程语言：[TypeScript 4.x](https://www.typescriptlang.org/zh/) + [JavaScript](https://www.javascript.com/)
- 构建工具：[Vite 2.x](https://cn.vitejs.dev/)
- 前端框架：[Vue 3.x](https://v3.cn.vuejs.org/)
- 路由工具：[Vue Router 4.x](https://next.router.vuejs.org/zh/index.html)
- 状态管理：[Vuex 4.x](https://next.vuex.vuejs.org/)
- UI 框架：[Element Plus](https://element-plus.org/#/zh-CN)
- CSS 预编译：[Stylus](https://stylus-lang.com/) / [Sass](https://sass.bootcss.com/documentation) / [Less](http://lesscss.cn/)
- HTTP 工具：[Axios](https://axios-http.com/)
- Git Hook 工具：[husky](https://typicode.github.io/husky/#/) + [lint-staged](https://github.com/okonet/lint-staged)
- 代码规范：[EditorConfig](http://editorconfig.org) + [Prettier](https://prettier.io/) + [ESLint](https://eslint.org/) + [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript#translation)
- 提交规范：[Commitizen](http://commitizen.github.io/cz-cli/) + [Commitlint](https://commitlint.js.org/#/)
- 单元测试：[vue-test-utils](https://next.vue-test-utils.vuejs.org/) + [jest](https://jestjs.io/) + [vue-jest](https://github.com/vuejs/vue-jest) + [ts-jest](https://kulshekhar.github.io/ts-jest/)
- 自动部署：[GitHub Actions](https://docs.github.com/cn/actions/learn-github-actions)

## 架构搭建

请确保你的电脑上成功安装 Node.js，本项目使用 Vite 构建工具，**需要 Node.js 版本 >= 12.0.0**。

查看 Node.js 版本：

```sh
node -v
复制代码
```

建议将 Node.js 升级到最新的稳定版本：

```bash
# 使用 nvm 安装最新稳定版 Node.js
nvm install stable
复制代码
```

### 使用 Vite 快速初始化项目雏形

- 使用 NPM：

  ```bash
  npm init @vitejs/app
  复制代码
  ```

- 使用 Yarn：

  ```bash
  yarn create @vitejs/app
  复制代码
  ```

然后按照终端提示完成以下操作：

1. 输入项目名称

   例如：本项目名称 **vite-vue3-starter**

   ![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/959dc45b86ca4066a1bcece8de88dc8d~tplv-k3u1fbpfcp-zoom-1.image)

2. 选择模板

   本项目需要使用 Vue3 + TypeScript，所以我们选择 `vue-ts`，会自动安装 Vue3 和 TypeScript。

   ![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/176e9cfb0f4545fc8d6ff8b5eb9422a2~tplv-k3u1fbpfcp-zoom-1.image)

   ![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7126a1dc802d411ab375289ac827b71e~tplv-k3u1fbpfcp-zoom-1.image)

   你还可以通过附加的命令行选项直接指定项目名和模板，本项目要构建 Vite + Vue3 + TypeScript 项目，则运行：

   ```bash
   # npm 6.x
   npm init @vitejs/app vite-vue3-starter --template vue-ts
   
   # npm 7+（需要额外的双横线）
   npm init @vitejs/app vite-vue3-starter -- --template vue-ts
   
   # yarn
   yarn create @vitejs/app vite-vue3-starter --template vue-ts
   复制代码
   ```

3. 安装依赖

   ```bash
   npm install
   复制代码
   ```

4. 启动项目

   ```bash
   npm run dev
   复制代码
   ```

   ![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3afd23c1469a45e895bb488eac45adfe~tplv-k3u1fbpfcp-zoom-1.image)

   如上图，表示 Vite + Vue3 + TypeScript 简单的项目骨架搭建完毕，下面我们来为这个项目集成 Vue Router、Vuex、Element Plus、Axios、Stylus/Sass/Less。

### 修改 Vite 配置文件

Vite 配置文件 `vite.config.ts` 位于根目录下，项目启动时会自动读取。

本项目先做一些简单配置，例如：设置 `@` 指向 `src` 目录、 服务启动端口、打包路径、代理等。

关于 Vite 更多配置项及用法，请查看 Vite 官网 [vitejs.dev/config/](https://vitejs.dev/config/) 。

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
// 如果编辑器提示 path 模块找不到，则可以安装一下 @types/node -> npm i @types/node -D
import { resolve } from 'path'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src') // 设置 `@` 指向 `src` 目录
    }
  },
  base: './', // 设置打包路径
  server: {
    port: 4000, // 设置服务启动端口号
    open: true, // 设置服务启动时是否自动打开浏览器
    cors: true // 允许跨域

    // 设置代理，根据我们项目实际情况配置
    // proxy: {
    //   '/api': {
    //     target: 'http://xxx.xxx.xxx.xxx:8000',
    //     changeOrigin: true,
    //     secure: false,
    //     rewrite: (path) => path.replace('/api/', '/')
    //   }
    // }
  }
})
复制代码
```

### 规范目录结构

```
├── publish/
└── src/
    ├── assets/                    // 静态资源目录
    ├── common/                    // 通用类库目录
    ├── components/                // 公共组件目录
    ├── router/                    // 路由配置目录
    ├── store/                     // 状态管理目录
    ├── style/                     // 通用 CSS 目录
    ├── utils/                     // 工具函数目录
    ├── views/                     // 页面组件目录
    ├── App.vue
    ├── main.ts
    ├── shims-vue.d.ts
├── tests/                         // 单元测试目录
├── index.html
├── tsconfig.json                  // TypeScript 配置文件
├── vite.config.ts                 // Vite 配置文件
└── package.json
复制代码
```

### 集成路由工具 Vue Router

1. 安装支持 Vue3 的路由工具 vue-router@4

   ```bash
   npm i vue-router@4
   复制代码
   ```

2. 创建 `src/router/index.ts` 文件

   在 `src` 下创建 `router` 目录，然后在 `router` 目录里新建 `index.ts` 文件：

   ```
    └── src/
        ├── router/
            ├── index.ts  // 路由配置文件
   复制代码
   ```

   ```ts
   import {
     createRouter,
     createWebHashHistory,
     RouteRecordRaw
   } from 'vue-router'
   import Home from '@/views/home.vue'
   import Vuex from '@/views/vuex.vue'
   
   const routes: Array<RouteRecordRaw> = [
     {
       path: '/',
       name: 'Home',
       component: Home
     },
     {
       path: '/vuex',
       name: 'Vuex',
       component: Vuex
     },
     {
       path: '/axios',
       name: 'Axios',
       component: () => import('@/views/axios.vue') // 懒加载组件
     }
   ]
   
   const router = createRouter({
     history: createWebHashHistory(),
     routes
   })
   
   export default router
   复制代码
   ```

   根据本项目路由配置的实际情况，你需要在 `src` 下创建 `views` 目录，用来存储页面组件。

   我们在 `views` 目录下创建 `home.vue` 、`vuex.vue` 、`axios.vue`。

3. 在 `main.ts` 文件中挂载路由配置

   ```ts
   import { createApp } from 'vue'
   import App from './App.vue'
   
   import router from './router/index'
   
   createApp(App).use(router).mount('#app')
   复制代码
   ```

### 集成状态管理工具 Vuex

1. 安装支持 Vue3 的状态管理工具 vuex@next

   ```bash
   npm i vuex@next
   复制代码
   ```

2. 创建 `src/store/index.ts` 文件

   在 `src` 下创建 `store` 目录，然后在 `store` 目录里新建 `index.ts` 文件：

   ```
   └── src/
       ├── store/
           ├── index.ts  // store 配置文件
   复制代码
   ```

   ```ts
   import { createStore } from 'vuex'
   
   const defaultState = {
     count: 0
   }
   
   // Create a new store instance.
   export default createStore({
     state() {
       return defaultState
     },
     mutations: {
       increment(state: typeof defaultState) {
         state.count++
       }
     },
     actions: {
       increment(context) {
         context.commit('increment')
       }
     },
     getters: {
       double(state: typeof defaultState) {
         return 2 * state.count
       }
     }
   })
   复制代码
   ```

3. 在 `main.ts` 文件中挂载 Vuex 配置

   ```ts
   import { createApp } from 'vue'
   import App from './App.vue'
   
   import store from './store/index'
   
   createApp(App).use(store).mount('#app')
   复制代码
   ```

### 集成 UI 框架 Element Plus

1. 安装支持 Vue3 的 UI 框架 Element Plus

   ```bash
   npm i element-plus
   复制代码
   ```

2. 在 `main.ts` 文件中挂载 Element Plus

   ```ts
   import { createApp } from 'vue'
   import App from './App.vue'
   
   import ElementPlus from 'element-plus'
   import 'element-plus/lib/theme-chalk/index.css'
   
   createApp(App).use(ElementPlus).mount('#app')
   复制代码
   ```

### 集成 HTTP 工具 Axios

1. 安装 Axios（Axios 跟 Vue 版本没有直接关系，安装最新即可）

   ```bash
   npm i axios
   复制代码
   ```

2. 配置 Axios

   > 为了使项目的目录结构合理且规范，我们在 `src` 下创建 `utils` 目录来存储我们常用的工具函数。

   Axios 作为 HTTP 工具，我们在 `utils` 目录下创建 `axios.ts` 作为 Axios 配置文件：

   ```
   └── src/
       ├── utils/
           ├── axios.ts  // Axios 配置文件
   复制代码
   ```

   ```ts
   import Axios from 'axios'
   import { ElMessage } from 'element-plus'
   
   const baseURL = 'https://api.github.com'
   
   const axios = Axios.create({
     baseURL,
     timeout: 20000 // 请求超时 20s
   })
   
   // 前置拦截器（发起请求之前的拦截）
   axios.interceptors.request.use(
     (response) => {
       /**
        * 根据你的项目实际情况来对 config 做处理
        * 这里对 config 不做任何处理，直接返回
        */
       return response
     },
     (error) => {
       return Promise.reject(error)
     }
   )
   
   // 后置拦截器（获取到响应时的拦截）
   axios.interceptors.response.use(
     (response) => {
       /**
        * 根据你的项目实际情况来对 response 和 error 做处理
        * 这里对 response 和 error 不做任何处理，直接返回
        */
       return response
     },
     (error) => {
       if (error.response && error.response.data) {
         const code = error.response.status
         const msg = error.response.data.message
         ElMessage.error(`Code: ${code}, Message: ${msg}`)
         console.error(`[Axios Error]`, error.response)
       } else {
         ElMessage.error(`${error}`)
       }
       return Promise.reject(error)
     }
   )
   
   export default axios
   复制代码
   ```

3. 使用 Axios
    在需要使用 Axios 文件里，引入 Axios 配置文件，参考如下：

   ```html
   <template></template>
   <script lang="ts">
     import { defineComponent } from 'vue'
     import axios from '../utils/axios'
   
     export default defineComponent({
       setup() {
         axios
           .get('/users/XPoet')
           .then((res) => {
             console.log('res: ', res)
           })
           .catch((err) => {
             console.log('err: ', err)
           })
       }
     })
   </script>
   复制代码
   ```

### 集成 CSS 预编译器 Stylus/Sass/Less

本项目使用 CSS 预编译器 Stylus，直接安装为开发依赖即可。Vite 内部已帮我们集成了相关的 loader，不需要额外配置。同理，你也可以使用 Sass 或 Less 等。

1. 安装

   ```bash
   npm i stylus -D
   # or
   npm i sass -D
   npm i less -D
   复制代码
   ```

2. 使用

   ```html
   <style lang="stylus">
     ...
   </style>
   复制代码
   ```

至此，一个基于 TypeScript + Vite + Vue3 + Vue Router + Vuex + Element Plus + Axios + Stylus/Sass/Less 的前端项目开发环境搭建完毕，项目 Demo 托管在 [GitHub 仓库](https://github.com/XPoet/vite-vue3-starter)，需要的同学可以去下载下来，参考学习。

下面我们来打磨这个项目，增加代码规范约束、提交规范约束、单元测试、自动部署等，让其更完善、更健壮。

## 代码规范

随着前端应用逐渐变得大型化和复杂化，在同一个项目中有多个人员参与时，每个人的前端能力程度不等，他们往往会用不同的编码风格和习惯在项目中写代码，长此下去，势必会让项目的健壮性越来越差。解决这些问题，理论上讲，口头约定和代码审查都可以，但是这种方式无法实时反馈，而且沟通成本过高，不够灵活，更关键的是无法把控。不以规矩，不能成方圆，我们不得不在项目使用一些工具来约束代码规范。

本文讲解如何使用 **EditorConfig + Prettier + ESLint** 组合来实现代码规范化。

这样做带来好处：

- 解决团队之间代码不规范导致的可读性差和可维护性差的问题。
- 解决团队成员不同编辑器导致的编码规范不统一问题。
- 提前发现代码风格问题，给出对应规范提示，及时修复。
- 减少代码审查过程中反反复复的修改过程，节约时间。
- 自动格式化，统一编码风格，从此和脏乱差的代码说再见。

### 集成 EditorConfig 配置

EditorConfig 有助于为不同 IDE 编辑器上处理同一项目的多个开发人员维护一致的编码风格。

官网：[editorconfig.org](http://editorconfig.org)

在项目根目录下增加 `.editorconfig` 文件：

```bash
# Editor configuration, see http://editorconfig.org

# 表示是最顶层的 EditorConfig 配置文件
root = true

[*] # 表示所有文件适用
charset = utf-8 # 设置文件字符集为 utf-8
indent_style = space # 缩进风格（tab | space）
indent_size = 2 # 缩进大小
end_of_line = lf # 控制换行类型(lf | cr | crlf)
trim_trailing_whitespace = true # 去除行首的任意空白字符
insert_final_newline = true # 始终在文件末尾插入一个新行

[*.md] # 表示仅 md 文件适用以下规则
max_line_length = off
trim_trailing_whitespace = false
复制代码
```

注意：

- VSCode 使用 EditorConfig 需要去插件市场下载插件 **EditorConfig for VS Code** 。

  ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- JetBrains 系列（WebStorm、IntelliJ IDEA 等）则不用额外安装插件，可直接使用 EditorConfig 配置。

### 集成 Prettier 配置

Prettier 是一款强大的代码格式化工具，支持 JavaScript、TypeScript、CSS、SCSS、Less、JSX、Angular、Vue、GraphQL、JSON、Markdown 等语言，基本上前端能用到的文件格式它都可以搞定，是当下最流行的代码格式化工具。

官网：[prettier.io/](https://prettier.io/)

1. 安装 Prettier

   ```bash
   npm i prettier -D
   复制代码
   ```

2. 创建 Prettier 配置文件

   Prettier 支持多种格式的[配置文件](https://prettier.io/docs/en/configuration.html)，比如 `.json`、`.yml`、`.yaml`、`.js`等。

   在本项目根目录下创建 `.prettierrc` 文件。

3. 配置 `.prettierrc`

   在本项目中，我们进行如下简单配置，关于更多配置项信息，请前往官网查看 [Prettier-Options](https://prettier.io/docs/en/options.html) 。

   ```json
   {
     "useTabs": false,
     "tabWidth": 2,
     "printWidth": 100,
     "singleQuote": true,
     "trailingComma": "none",
     "bracketSpacing": true,
     "semi": false
   }
   复制代码
   ```

4. Prettier 安装且配置好之后，就能使用命令来格式化代码

   ```bash
   # 格式化所有文件（. 表示所有文件）
   npx prettier --write .
   复制代码
   ```

注意：

- VSCode 编辑器使用 Prettier 配置需要下载插件 **Prettier - Code formatter** 。

  ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- JetBrains 系列编辑器（WebStorm、IntelliJ IDEA 等）则不用额外安装插件，可直接使用 Prettier 配置。

Prettier 配置好以后，在使用 VSCode 或 WebStorm 等编辑器的格式化功能时，编辑器就会按照 Prettier 配置文件的规则来进行格式化，避免了因为大家编辑器配置不一样而导致格式化后的代码风格不统一的问题。

### 集成 ESLint 配置

[ESLint](https://github.com/eslint/eslint) 是一款用于查找并报告代码中问题的工具，并且支持部分问题自动修复。其核心是通过对代码解析得到的 AST（Abstract Syntax Tree 抽象语法树）进行模式匹配，来分析代码达到检查代码质量和风格问题的能力。

正如前面我们提到的因团队成员之间编程能力和编码习惯不同所造成的代码质量问题，我们使用 ESLint 来解决，一边写代码一边查找问题，如果发现错误，就给出规则提示，并且自动修复，长期下去，可以促使团队成员往同一种编码风格靠拢。

1. 安装 ESLint

   可以全局或者本地安装，作者推荐本地安装（只在当前项目中安装）。

   ```bash
   npm i eslint -D
   复制代码
   ```

2. 配置 ESLint

   ESLint 安装成功后，执行 `npx eslint --init`，然后按照终端操作提示完成一系列设置来创建配置文件。

   - How would you like to use ESLint? （你想如何使用 ESLint?）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     我们这里选择 **To check syntax, find problems, and enforce code style（检查语法、发现问题并强制执行代码风格）**

   - What type of modules does your project use?（你的项目使用哪种类型的模块?）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     我们这里选择 **JavaScript modules (import/export)**

   - Which framework does your project use? （你的项目使用哪种框架?）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     我们这里选择 **Vue.js**

   - Does your project use TypeScript?（你的项目是否使用 TypeScript？）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     我们这里选择 **Yes**

   - Where does your code run?（你的代码在哪里运行?）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     我们这里选择 **Browser 和 Node**（按空格键进行选择，选完按回车键确定）

   - How would you like to define a style for your project?（你想怎样为你的项目定义风格？）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     我们这里选择 **Use a popular style guide（使用一种流行的风格指南）**

   - Which style guide do you want to follow?（你想遵循哪一种风格指南?）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     我们这里选择 **Airbnb: [github.com/airbnb/java…](https://github.com/airbnb/javascript)**

     ESLint 为我们列出了三种社区流行的 JavaScript 风格指南，分别是 Airbnb、Standard、Google。

     这三份风格指南都是由众多大佬根据多年开发经验编写，足够优秀，全球很多大小公司都在使用。我们选用 **GitHub 上 star 最多的 Airbnb**，免去繁琐的配置 ESLint 规则时间，然后让团队成员去学习 Airbnb JavaScript 风格指南即可。

     此时，我们在 ESLint 配置了 Airbnb JavaScript 规则，在编码时，所有不符合 Airbnb 风格的代码，编辑器都会给出提示，并且可以自动修复。

     **这里作者不建议大家去自由配置 ESLint 规则，相信我，这三份 JavaScript 代码风格指南值得我们反复学习，掌握后，编程能力能上一大台阶。**

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     - [JavaScript Standard Style](https://github.com/standard/standard)

       ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     - [Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)

   - What format do you want your config file to be in?（你希望你的配置文件是什么格式?）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     我们这里选择 **JavaScript**

   - Would you like to install them now with npm?（你想现在就用 NPM 安装它们吗?）

     ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     根据上面的选择，ESLint 会自动去查找缺失的依赖，我们这里选择 **Yes**，使用 NPM 下载安装这些依赖包。

     注意：如果自动安装依赖失败，那么需要手动安装

     ```bash
     npm i @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-config-airbnb-base eslint-plugin-import eslint-plugin-vue -D
     复制代码
     ```

3. ESLint 配置文件 `.eslintrc.js`

   在**上一步**操作完成后，会在项目根目录下自动生成 `.eslintrc.js` 配置文件：

   ```js
   module.exports = {
     env: {
       browser: true,
       es2021: true,
       node: true
     },
     extends: ['plugin:vue/essential', 'airbnb-base'],
     parserOptions: {
       ecmaVersion: 12,
       parser: '@typescript-eslint/parser',
       sourceType: 'module'
     },
     plugins: ['vue', '@typescript-eslint'],
     rules: {}
   }
   复制代码
   ```

   根据项目实际情况，如果我们有额外的 ESLint 规则，也在此文件中追加。

注意：

- VSCode 使用 ESLint 配置文件需要去插件市场下载插件 **ESLint** 。

  ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- JetBrains 系列（WebStorm、IntelliJ IDEA 等）则不用额外安装插件。

配置好以后，我们在 VSCode 或 WebStorm 等编辑器中开启 ESLin，写代码时，ESLint 就会按照我们配置的规则来进行实时代码检查，发现问题会给出对应错误提示和修复方案。

如图：

- VSCode ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)
- WebStorm ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

虽然，现在编辑器已经给出错误提示和修复方案，但需要我们一个一个去点击修复，还是挺麻烦的。很简单，我们只需设置编辑器保存文件时自动执行 `eslint --fix` 命令进行代码风格修复。

- VSCode 在 `settings.json` 设置文件中，增加以下代码：

  ```js
   "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
   }
  复制代码
  ```

- WebStorm 打开设置窗口，按如下操作，最后点击 `Apply` -> `OK`。 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

### 解决 Prettier 和 ESLint 的冲突

通常大家会在项目中根据实际情况添加一些额外的 ESLint 和 Prettier 配置规则，难免会存在规则冲突情况。

本项目中的 ESLint 配置中使用了 Airbnb JavaScript 风格指南校验，其规则之一是*代码结束后面要加分号*，而我们在 Prettier 配置文件中加了*代码结束后面不加分号*的配置项，这样就有冲突了，会出现用 Prettier 格式化后的代码，ESLint 检测到格式有问题的，从而抛出错误提示。

解决两者冲突问题，需要用到 **eslint-plugin-prettier** 和 **eslint-config-prettier**。

- `eslint-plugin-prettier` 将 Prettier 的规则设置到 ESLint 的规则中。
- `eslint-config-prettier` 关闭 ESLint 中与 Prettier 中会发生冲突的规则。

最后形成优先级：`Prettier 配置规则` > `ESLint 配置规则`。

- 安装插件

  ```bash
  npm i eslint-plugin-prettier eslint-config-prettier -D
  复制代码
  ```

- 在 `.eslintrc.js` 添加 prettier 插件

  ```js
  module.exports = {
    ...
    extends: [
      'plugin:vue/essential',
      'airbnb-base',
      'plugin:prettier/recommended' // 添加 prettier 插件
    ],
    ...
  }
  复制代码
  ```

这样，我们在执行 `eslint --fix` 命令时，ESLint 就会按照 Prettier 的配置规则来格式化代码，轻松解决二者冲突问题。

### 集成 husky 和 lint-staged

我们在项目中已集成 ESLint 和 Prettier，在编码时，这些工具可以对我们写的代码进行实时校验，在一定程度上能有效规范我们写的代码，但团队可能会有些人觉得这些条条框框的限制很麻烦，选择视“提示”而不见，依旧按自己的一套风格来写代码，或者干脆禁用掉这些工具，开发完成就直接把代码提交到了仓库，日积月累，ESLint 也就形同虚设。

所以，我们还需要做一些限制，让没通过 ESLint 检测和修复的代码禁止提交，从而保证仓库代码都是符合规范的。

为了解决这个问题，我们需要用到 Git Hook，在本地执行 `git commit` 的时候，就对所提交的代码进行 ESLint 检测和修复（即执行 `eslint --fix`），如果这些代码没通过 ESLint 规则校验，则禁止提交。

实现这一功能，我们借助 [husky](https://github.com/typicode/husky) + [lint-staged](https://github.com/okonet/lint-staged) 。

> [husky](https://github.com/typicode/husky) —— Git Hook 工具，可以设置在 git 各个阶段（`pre-commit`、`commit-msg`、`pre-push` 等）触发我们的命令。
>  [lint-staged](https://github.com/okonet/lint-staged) —— 在 git 暂存的文件上运行 linters。

#### 配置 husky

- 自动配置（推荐）

  使用 `husky-init` 命令快速在项目初始化一个 husky 配置。

  ```bash
  npx husky-init && npm install
  复制代码
  ```

  这行命令做了四件事：

  1. 安装 husky 到开发依赖 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)
  2. 在项目根目录下创建 `.husky` 目录 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)
  3. 在 `.husky` 目录创建 `pre-commit` hook，并初始化 `pre-commit` 命令为 `npm test` ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)
  4. 修改 `package.json` 的 `scripts`，增加 `"prepare": "husky install"` ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 手动配置（不推荐，懒是程序员第一生产力）

  1. 安装 husky

     ```bash
     npm i husky -D
     复制代码
     ```

  2. 创建 Git hooks

     ```bash
     npx husky install
     复制代码
     ```

     该命令做了两件事：

     - 在项目根目录下创建 `.husky` 目录 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)
     - 在 `.husky` 目录创建 `pre-commit` hook，并初始化 `pre-commit` 命令为 `npm test` ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  3. 手动修改 `package.json` 的 `scripts`，增加 `"prepare": "husky install"` ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

> **特别注意：本项目使用 husky 6.x 版本，6.x 版本配置方式跟之前的版本有较大差异。目前网上大部分有关 husky 的教程都是 6 以前的版本 ，跟本文教程不太一样，当发现配置方法不一致时，一切以 [husky 官网](https://typicode.github.io/husky/#/?id=usage)为准。**

到这里，husky 配置完毕，现在我们来使用它：

husky 包含很多 `hook`（钩子），常用有：`pre-commit`、`commit-msg`、`pre-push`。这里，我们使用 `pre-commit` 来触发 ESLint 命令。

修改 `.husky/pre-commit` hook 文件的触发命令：

```bash
eslint --fix ./src --ext .vue,.js,.ts
复制代码
```

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

上面这个 `pre-commit` hook 文件的作用是：当我们执行 `git commit -m "xxx"` 时，会先对 `src` 目录下所有的 `.vue`、`.js`、`.ts ` 文件执行 `eslint --fix` 命令，如果 ESLint 通过，成功 `commit`，否则终止 `commit`。

但是又存在一个问题：有时候我们明明只改动了一两个文件，却要对所有的文件执行 `eslint --fix`。假如这是一个历史项目，我们在中途配置了 ESLint 规则，那么在提交代码时，也会对其他未修改的“历史”文件都进行检查，可能会造成大量文件出现 ESLint 错误，显然不是我们想要的结果。

我们要做到只用 ESLint 修复自己此次写的代码，而不去影响其他的代码。所以我们还需借助一个神奇的工具 **lint-staged** 。

#### 配置 lint-staged

lint-staged 这个工具一般结合 husky 来使用，它可以让 husky 的 `hook` 触发的命令只作用于 `git add`那些文件（即 git 暂存区的文件），而不会影响到其他文件。

接下来，我们使用 lint-staged 继续优化项目。

1. 安装 lint-staged

   ```bash
   npm i lint-staged -D
   复制代码
   ```

2. 在 `package.json`里增加 lint-staged 配置项

   ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

   ```json
   "lint-staged": {
     "*.{vue,js,ts}": "eslint --fix"
   },
   复制代码
   ```

   这行命令表示：只对 git 暂存区的 `.vue`、`.js`、`.ts` 文件执行 `eslint --fix`。

3. 修改 `.husky/pre-commit` hook 的触发命令为：`npx lint-staged`

   ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

至此，husky 和 lint-staged 组合配置完成。

现在我们提交代码时就会变成这样：

假如我们修改了 `scr` 目录下的 `test-1.js`、`test-2.ts` 和 `test-3.md` 文件，然后 `git add ./src/`，最后 `git commit -m "test..."`，这时候就会只对 `test-1.js`、`test-2.ts` 这两个文件执行 `eslint --fix`。如果 ESLint 通过，成功提交，否则终止提交。从而保证了我们提交到 Git 仓库的代码都是规范的。

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 提交前 `test-1.js`、`test-2.ts` ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)
- 提交后 `test-1.js`、`test-2.ts` 自动修复代码格式 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

无论写代码还是做其他事情，都应该用长远的眼光来看，刚开始使用 ESint 的时候可能会有很多问题，改起来也很费时费力，只要坚持下去，代码质量和开发效率都会得到提升，前期的付出都是值得的。

这些工具并不是必须的，没有它们你同样可以可以完成功能开发，但是利用好这些工具，你可以写出更高质量的代码。特别是一些刚刚接触的人，可能会觉得麻烦而放弃使用这些工具，失去了一次提升编程能力的好机会。

> 本项目完整的代码托管在 [GitHub 仓库](https://github.com/XPoet/vite-vue3-starter)，有需要的同学可以去下载下来，参考学习。
>  [点亮小星星 🌟 支持作者~](https://github.com/XPoet/vite-vue3-starter)

## 提交规范

前面我们已经统一代码规范，并且在提交代码时进行强约束来保证仓库代码质量。多人协作的项目中，在提交代码这个环节，也存在一种情况：不能保证每个人对提交信息的准确描述，因此会出现提交信息紊乱、风格不一致的情况。

如果 `git commit` 的描述信息精准，在后期维护和 Bug 处理时会变得有据可查，项目开发周期内还可以根据规范的提交信息快速生成开发日志，从而方便我们追踪项目和把控进度。

这里，我们使用社区最流行、最知名、最受认可的 Angular 团队提交规范。

先看看 [Angular 项目的提交记录](https://github.com/angular/angular/commits/master)：

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

如上图，可以看出这些提交信息都是有固定格式的，下面我们来学习 Angular 规范的 commit message 格式。

### commit message 格式规范

commit message 由 Header、Body、Footer 组成。

```
<Header>

<Body>

<Footer>
复制代码
```

#### Header

Header 部分包括三个字段 type（必需）、scope（可选）和 subject（必需）。

```
<type>(<scope>): <subject>
复制代码
```

##### type

type 用于说明 commit 的提交类型（必须是以下几种之一）。

| 值       | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| feat     | 新增一个功能                                                 |
| fix      | 修复一个 Bug                                                 |
| docs     | 文档变更                                                     |
| style    | 代码格式（不影响功能，例如空格、分号等格式修正）             |
| refactor | 代码重构                                                     |
| perf     | 改善性能                                                     |
| test     | 测试                                                         |
| build    | 变更项目构建或外部依赖（例如 scopes: webpack、gulp、npm 等） |
| ci       | 更改持续集成软件的配置文件和 package 中的 scripts 命令，例如 scopes: Travis, Circle 等 |
| chore    | 变更构建流程或辅助工具                                       |
| revert   | 代码回退                                                     |

##### scope

scope 用于指定本次 commit 影响的范围。scope 依据项目而定，例如在业务项目中可以依据菜单或者功能模块划分，如果是组件库开发，则可以依据组件划分。（scope 可省略）

##### subject

subject 是本次 commit 的简洁描述，长度约定在 50 个字符以内，通常遵循以下几个规范：

- 用动词开头，第一人称现在时表述，例如：change 代替 changed 或 changes
- 第一个字母小写
- 结尾不加句号（.）

#### Body

body 是对本次 commit 的详细描述，可以分成多行。（body 可省略）

跟 subject 类似，用动词开头，body 应该说明修改的原因和更改前后的行为对比。

#### Footer

如果本次提交的代码是突破性的变更或关闭缺陷，则 Footer 必需，否则可以省略。

- 突破性的变更

  当前代码与上一个版本有突破性改变，则 Footer 以 BREAKING CHANGE 开头，后面是对变动的描述、以及变动的理由。

- 关闭缺陷

  如果当前提交是针对特定的 issue，那么可以在 Footer 部分填写需要关闭的单个 issue 或一系列 issues。

#### 参考例子

- feat

  ```
  feat(browser): onUrlChange event (popstate/hashchange/polling)
  
  Added new event to browser:
  - forward popstate event if available
  - forward hashchange event if popstate not available
  - do polling when neither popstate nor hashchange available
  
  Breaks $browser.onHashChange, which was removed (use onUrlChange instead)
  复制代码
  ```

- fix

  ```
  fix(compile): couple of unit tests for IE9
  
  Older IEs serialize html uppercased, but IE9 does not...
  Would be better to expect case insensitive, unfortunately jasmine does
  not allow to user regexps for throw expectations.
  
  Closes #392
  Breaks foo.bar api, foo.baz should be used instead
  复制代码
  ```

- style

  ```
  style(location): add couple of missing semi colons
  复制代码
  ```

- chore

  ```
  chore(release): v3.4.2
  复制代码
  ```

#### 规范 commit message 的好处

- 首行就是简洁实用的关键信息，方便在 git history 中快速浏览。
- 具有更加详细的 body 和 footer，可以清晰的看出某次提交的目的和影响。
- 可以通过 type 过滤出想要查找的信息，也可以通过关键字快速查找相关提交。
- 可以直接从 commit 生成 change log。

### 集成 Commitizen 实现规范提交

上面介绍了 Angular 规范提交的格式，初次接触的同学咋一看可能会觉得复杂，其实不然，如果让大家在 `git commit` 的时候严格按照上面的格式来写，肯定是有压力的，首先得记住不同的类型到底是用来定义什么，subject 怎么写，body 怎么写，footer 要不要写等等问题，懒才是程序员第一生产力，为此我们使用 Commitizen 工具来帮助我们自动生成 commit message 格式，从而实现规范提交。

> Commitizen 是一个帮助撰写规范 commit message 的工具。它有一个命令行工具 cz-cli。

#### 安装 Commitizen

```
npm install commitizen -D
复制代码
```

#### 初始化项目

成功安装 Commitizen 后，我们用 **cz-conventional-changelog** 适配器来初始化项目：

```
npx commitizen init cz-conventional-changelog --save-dev --save-exact
复制代码
```

这行命令做了两件事：

- 安装 cz-conventional-changelog 到开发依赖（devDependencies）

- 在 

  ```
  package.json
  ```

   中增加了 

  ```
  config.commitizen
  ```

  ```
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
  复制代码
  ```

  

#### 使用 Commitizen

以前我们提交代码都是 `git commit -m "xxx"`，现在改为 `git cz`，然后按照终端操作提示，逐步填入信息，就能自动生成规范的 commit message。

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

最后，在 Git 提交历史中就能看到刚刚规范的提交记录了： ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

#### 自定义配置提交说明

从上面的截图可以看到，`git cz` 终端操作提示都是英文的，如果想改成中文的或者自定义这些配置选项，我们使用 **cz-customizable** 适配器。

##### cz-customizable 初始化项目

运行如下命令使用 cz-customizable 初始化项目，注意之前已经初始化过一次，这次再初始化，需要加 `--force` 覆盖。

```
npx commitizen init cz-customizable --save-dev --save-exact --force
复制代码
```

这行命令做了两件事：

- 安装 cz-customizable 到开发依赖（devDependencies）

  ```json
  "devDependencies": {
    ...
    "cz-customizable": "^6.3.0",
    ...
  },
  复制代码
  ```

- 修改 `package.json` 中的 `config.commitizen` 字段为：

  ```json
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-customizable"
    }
  }
  复制代码
  ```

##### 使用 cz-customizable

在项目根目录下创建 `.cz-config.js` 文件，然后按照官方提供的[示例](https://github.com/leoforfree/cz-customizable/blob/master/cz-config-EXAMPLE.js)来配置。

在本项目中我们修改成中文：

```js
module.exports = {
  // type 类型（定义之后，可通过上下键选择）
  types: [
    { value: 'feat', name: 'feat:     新增功能' },
    { value: 'fix', name: 'fix:      修复 bug' },
    { value: 'docs', name: 'docs:     文档变更' },
    { value: 'style', name: 'style:    代码格式（不影响功能，例如空格、分号等格式修正）' },
    { value: 'refactor', name: 'refactor: 代码重构（不包括 bug 修复、功能新增）' },
    { value: 'perf', name: 'perf:     性能优化' },
    { value: 'test', name: 'test:     添加、修改测试用例' },
    { value: 'build', name: 'build:    构建流程、外部依赖变更（如升级 npm 包、修改 webpack 配置等）' },
    { value: 'ci', name: 'ci:       修改 CI 配置、脚本' },
    { value: 'chore', name: 'chore:    对构建过程或辅助工具和库的更改（不影响源文件、测试用例）' },
    { value: 'revert', name: 'revert:   回滚 commit' }
  ],

  // scope 类型（定义之后，可通过上下键选择）
  scopes: [
    ['components', '组件相关'],
    ['hooks', 'hook 相关'],
    ['utils', 'utils 相关'],
    ['element-ui', '对 element-ui 的调整'],
    ['styles', '样式相关'],
    ['deps', '项目依赖'],
    ['auth', '对 auth 修改'],
    ['other', '其他修改'],
    // 如果选择 custom，后面会让你再输入一个自定义的 scope。也可以不设置此项，把后面的 allowCustomScopes 设置为 true
    ['custom', '以上都不是？我要自定义']
  ].map(([value, description]) => {
    return {
      value,
      name: `${value.padEnd(30)} (${description})`
    }
  }),

  // 是否允许自定义填写 scope，在 scope 选择的时候，会有 empty 和 custom 可以选择。
  // allowCustomScopes: true,

  // allowTicketNumber: false,
  // isTicketNumberRequired: false,
  // ticketNumberPrefix: 'TICKET-',
  // ticketNumberRegExp: '\\d{1,5}',


  // 针对每一个 type 去定义对应的 scopes，例如 fix
  /*
  scopeOverrides: {
    fix: [
      { name: 'merge' },
      { name: 'style' },
      { name: 'e2eTest' },
      { name: 'unitTest' }
    ]
  },
  */

  // 交互提示信息
  messages: {
    type: '确保本次提交遵循 Angular 规范！\n选择你要提交的类型：',
    scope: '\n选择一个 scope（可选）：',
    // 选择 scope: custom 时会出下面的提示
    customScope: '请输入自定义的 scope：',
    subject: '填写简短精炼的变更描述：\n',
    body:
      '填写更加详细的变更描述（可选）。使用 "|" 换行：\n',
    breaking: '列举非兼容性重大的变更（可选）：\n',
    footer: '列举出所有变更的 ISSUES CLOSED（可选）。 例如: #31, #34：\n',
    confirmCommit: '确认提交？'
  },

  // 设置只有 type 选择了 feat 或 fix，才询问 breaking message
  allowBreakingChanges: ['feat', 'fix'],

  // 跳过要询问的步骤
  // skipQuestions: ['body', 'footer'],

  // subject 限制长度
  subjectLimit: 100
  breaklineChar: '|', // 支持 body 和 footer
  // footerPrefix : 'ISSUES CLOSED:'
  // askForBreakingChangeFirst : true,
}
复制代码
```

建议大家结合项目实际情况来自定义配置提交规则，例如很多时候我们不需要写长描述，公司内部的代码仓库也不需要管理 issue，那么可以把询问 body 和 footer 的步骤跳过（在 `.cz-config.js` 中修改成 `skipQuestions: ['body', 'footer']`）。

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

### 集成 commitlint 验证提交规范

在“代码规范”章节，我们已经讲到过，尽管制定了规范，但在多人协作的项目中，总有些人依旧我行我素，因此提交代码这个环节，我们也增加一个限制：**只让符合 Angular 规范的 commit message 通过**，我们借助 @commitlint/config-conventional 和 @commitlint/cli 来实现。

#### 安装 commitlint

安装 @commitlint/config-conventional 和 @commitlint/cli

```bash
npm i @commitlint/config-conventional @commitlint/cli -D
复制代码
```

#### 配置 commitlint

- 创建 commitlint.config.js 文件 在项目根目录下创建 `commitlint.config.js` 文件，并填入以下内容：

  ```js
  module.exports = { extends: ['@commitlint/config-conventional'] }
  复制代码
  ```

  或直接使用快捷命令：

  ```bash
  echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
  复制代码
  ```

- 使用 husky 的 `commit-msg` hook 触发验证提交信息的命令
   我们使用 husky 命令在 `.husky` 目录下创建 `commit-msg` 文件，并在此执行 commit message 的验证命令。

  ```bash
  npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
  复制代码
  ```

  ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

#### commitlint 验证

- 不符合规范的提交信息
   如下图，提交信息 `test commitlint` 不符合规范，提交失败。 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)
- 符合规范的提交信息
   如下图，提交信息 `test: commitlint test` 符合规范，成功提交到仓库。 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

因为已在项目中集成 commitizen，建议大家用 `git cz` 来代替 `git commit` 提交代码，可以保证提交信息规范。

> 本项目完整的代码托管在 [GitHub 仓库](https://github.com/XPoet/vite-vue3-starter)，同学可以去下载下来，参考学习。
>  [点亮小星星 🌟 支持作者~](https://github.com/XPoet/vite-vue3-starter)

## 单元测试

单元测试是项目开发中一个非常重要的环节，完整的测试能为代码和业务提供质量保证，减少 Bug 的出现。

本章节将带领大家在 Vite + Vue3 + TypeScript 的项目中集成单元测试工具。

### 安装核心依赖

我们使用 Vue 官方提供的 **vue-test-utils** 和社区流行的测试工具 **jest** 来进行 Vue 组件的单元测试。

- **[vue-test-utils](https://github.com/vuejs/vue-test-utils-next)** The next iteration of Vue Test Utils. It targets Vue 3.
- **[jest](https://github.com/facebook/jest)** Delightful JavaScript Testing.
- **[vue-jest](https://github.com/vuejs/vue-jest)** Jest Vue transformer
- **[ts-jest](https://github.com/kulshekhar/ts-jest)** A Jest transformer with source map support that lets you use Jest to test projects written in TypeScript.

安装这些工具为开发依赖（devDependencies）：

```bash
npm i @vue/test-utils@next jest vue-jest@next ts-jest -D
复制代码
```

### 创建 jest 配置文件

在项目根目录下新建 `jest.config.js` 文件：

```js
module.exports = {
  moduleFileExtensions: ['vue', 'js', 'ts'],
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  transform: {
    '^.+\\.vue$': 'vue-jest', // vue 文件用 vue-jest 转换
    '^.+\\.ts$': 'ts-jest' // ts 文件用 ts-jest 转换
  },
  // 匹配 __tests__ 目录下的 .js/.ts 文件 或其他目录下的 xx.test.js/ts xx.spec.js/ts
  testRegex: '(/__tests__/.*|(\\.|/)(test|spec))\\.(ts)$'
}
复制代码
```

### 创建单元测试文件

在上面的 `jest.config.js` 文件中，我们配置只匹配 `__tests__` 目录下的任意 `.ts` 文件或其他目录下的 `xx.test.ts`/`xx.spec.ts` 文件进行单元测试。

这里，我们在项目根目录下创建 `tests` 目录来存储单元测试文件

```
├── src/
└── tests/                           // 单元测试目录
    ├── Test.spec.ts                 // Test 组件测试
复制代码
```

- `Test.vue`

```html
<template>
  <div class="test-container page-container">
    <div class="page-title">Unit Test Page</div>
    <p>count is: {{ count }}</p>
    <button @click="increment">increment</button>
  </div>
</template>

<script lang="ts">
  import { defineComponent, ref } from 'vue'

  export default defineComponent({
    name: 'Vuex',
    setup() {
      const count = ref<number>(0)
      const increment = () => {
        count.value += 1
      }
      return { count, increment }
    }
  })
</script>
复制代码
```

- `Test.spec.ts`

  ```ts
  import { mount } from '@vue/test-utils'
  import Test from '../src/views/Test.vue'
  
  test('Test.vue', async () => {
    const wrapper = mount(Test)
    expect(wrapper.html()).toContain('Unit Test Page')
    expect(wrapper.html()).toContain('count is: 0')
    await wrapper.find('button').trigger('click')
    expect(wrapper.html()).toContain('count is: 1')
  })
  复制代码
  ```

### 集成 @types/jest

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

如上图，我们使用 VSCode / WebStrom / IDEA 等编辑器时，在单元测试文件中，IDE 会提示某些方法不存在（如 `test`、`describe`、`it`、`expect`等），安装 @types/jest 即可解决。

```
npm i @types/jest -D
复制代码
```

TypeScript 的编译器也会提示 jest 的方法和类型找不到，我们还需把 @types/jest 添加根目录下的 `ts.config.json`（TypeScript 配置文件）中：

```json
{
  "compilerOptions": {
    ...
    "types": ["vite/client", "jest"]
  },
}
复制代码
```

### 添加 eslint-plugin-jest

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

因为我们在项目中集成了 ESLint，如上图很明显是没通过 ESLint 规则检验。因此，我们还需要在 ESLint 中增加 **eslint-plugin-jest** 插件来解除对 jest 的校验。

- 安装 eslint-plugin-jest

  ```bash
  npm i eslint-plugin-jest -D
  复制代码
  ```

- 添加 eslint-plugin-jest 到 ESLint 配置文件 `.eslintrc.js` 中

  ```js
  module.exports = {
    ...
    extends: [
      ...
      'plugin:jest/recommended'
    ],
    ...
  }
  复制代码
  ```

现在，我们的单元测试代码就不会有错误提示信息了 ؏؏☝ᖗ 乛 ◡ 乛 ᖘ☝؏؏

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

### 执行单元测试

在根目录下 `package.json` 文件的 `scripts` 中，添加一条单元测试命令： `"test": "jest"`。

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

执行命令 `npm run test` 即可进行单元测试，jest 会根据 `jest.config.js` 配置文件去查找 `__tests__` 目录下的 `.ts` 文件或其他任意目录下的 `.spec.ts` 和 `.test.ts` 文件，然后执行单元测试方法。

> 你可以在 `jest.config.js` 配置文件中，自由配置单元测试文件的目录。

- 单元测试全部通过时的终端显示信息 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)
- 单元测试未全部通过时的终端显示信息 ![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

当单元测试没有全部通过时，我们需要根据报错信息去优化对应组件的代码，进一步提高项目健壮性。但是写单元测试是件比较痛苦的事，我个人觉得也没必要全部组件都写单元测试，根据项目实际情况有针对性去写就行了。

### 单元测试约束

前面，我们使用 husky 在 Git 的 `pre-commit` 和 `commit-msg` 阶段分别约束代码风格规范和提交信息规范。这一步，我们在 `pre-push` 阶段进行单元测试，只有单元测试全部通过才让代码 `push` 到远端仓库，否则终止 `push`。

使用 husky 命令在 `.husky` 目录下自动创建 `pre-push` hook 文件，并在此执行单元测试命令 `npm run test`。

```
npx husky add .husky/pre-push "npm run test $1"
复制代码
```

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

现在，我们在 `git push` 时就能先进行单元测试了，只有单元测试全部通过，才能成功 `push`。

> 本项目完整的代码托管在 [GitHub 仓库](https://github.com/XPoet/vite-vue3-starter)，同学可以去下载下来，参考学习。
>  [点亮小星星 🌟 支持作者~](https://github.com/XPoet/vite-vue3-starter)

## 自动部署

到了这一步，我们已经在项目中集成**代码规范约束**、**提交信息规范约束**，**单元测试约束**，从而保证我们远端仓库（如 GitHub、GitLab、Gitee 仓库等）的代码都是高质量的。

本项目是要搭建一套规范的前端工程化环境，为此我们使用 CI（Continuous Integration 持续集成）来完成项目最后的部署工作。

常见的 CI 工具有 GitHub Actions、GitLab CI、Travis CI、Circle CI 等。

这里，我们使用 GitHub Actions。

### 什么是 GitHub Actions

GitHub Actions 是 GitHub 的持续集成服务，持续集成由很多操作组成，比如抓取代码、运行测试、登录远程服务器、发布到第三方服务等等，GitHub 把这些操作称为 actions。

### 配置 GitHub Actions

#### 创建 GitHub 仓库

因为 GitHub Actions 只对 GitHub 仓库有效，所以我们[创建 GitHub 仓库](https://github.com/new)来托管项目代码。

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

其中，我们用：

- `master` 分支存储项目源代码
- `gh-pages` 分支存储打包后的静态文件

> `gh-pages` 分支，是 GitHub Pages 服务的固定分支，可以通过 HTTP 的方式访问到这个分支的静态文件资源。

#### 创建 GitHub Token

创建一个有 **repo** 和 **workflow** 权限的 [GitHub Token](https://github.com/settings/tokens/new)

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

注意：新生成的 Token 只会显示一次，保存起来，后面要用到。如有遗失，重新生成即可。

![image](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

#### 在仓库中添加 secret

将上面新创建的 Token 添加到 GitHub 仓库的 `Secrets` 里，并将这个新增的 `secret` 命名为 `VUE3_DEPLOY` （名字无所谓，看你喜欢）。

步骤：仓库 -> `settings` -> `Secrets` -> `New repository secret`。

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/927922363c62497eb01ce72e59155278~tplv-k3u1fbpfcp-zoom-1.image)

> 新创建的 secret `VUE3_DEPLOY` 在 Actions 配置文件中要用到，两个地方需保持一致！

#### 创建 Actions 配置文件

1. 在项目根目录下创建 `.github` 目录。
2. 在 `.github` 目录下创建 `workflows` 目录。
3. 在 `workflows` 目录下创建 `deploy.yml` 文件。

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc47cbed18534ac5abdeb1ec2f0f9664~tplv-k3u1fbpfcp-zoom-1.image)

`deploy.yml` 文件的内容：

```yaml
name: deploy

on:
  push:
    branches: [master] # master 分支有 push 时触发

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Setup Node.js v14.x
        uses: actions/setup-node@v1
        with:
          node-version: '14.x'

      - name: Install
        run: npm install # 安装依赖

      - name: Build
        run: npm run build # 打包

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3 # 使用部署到 GitHub pages 的 action
        with:
          publish_dir: ./dist # 部署打包后的 dist 目录
          github_token: ${{ secrets.VUE3_DEPLOY }} # secret 名
          user_name: ${{ secrets.MY_USER_NAME }}
          user_email: ${{ secrets.MY_USER_EMAIL }}
          commit_message: Update Vite2.x + Vue3.x + TypeScript Starter # 部署时的 git 提交信息，自由填写
复制代码
```

### 自动部署触发原理

当有新提交的代码 `push` 到 GitHub 仓库时，就会触发 GitHub Actions，在 GitHub 服务器上执行 Action 配置文件里面的命令，例如：**安装依赖**、**项目打包**等，然后将打包好的静态文件部署到 GitHub Pages 上，最后，我们就能通过域名访问了。

> 🌏 通过域名 [vite-vue3-starter.xpoet.cn/](https://vite-vue3-starter.xpoet.cn/) 访问本项目

使用自动部署，我们只需专注于项目开发阶段，任何重复且枯燥的行为都交由程序去完成，懒才是程序员第一生产力。

事实上，自动部署只是 GitHub Actions 功能的冰山一角，GitHub Actions 能做的事还很多很多，大家感兴趣的话自行查阅。


