[TOC]





## 安装webpack

- 需要先在项目中npm init初始化一下，生成package.json
- 建议node版本安装到8.2以上

```
// webpack4中除了正常安装webpack之外，需要再单独安一个webpack-cli

npm i webpack webpack-cli -D
复制代码
```

★ npm i -D 是 npm install --save-dev 的简写，是指安装模块并保存到 package.json 的 devDependencies中，主要在开发环境中的依赖包

## 0配置了什么

webpack4可以支持0配置打包，这里所说的0配置又是什么呢？当然在开发者眼中0配置的东西，那根本是无法用的，因为不够智能，那么我们就来看看做到了哪些0配置

在使用webpack进行打包的时候，默认情况下会将src下的入口文件(index.js)进行打包

```
// node v8.2版本以后都会有一个npx
// npx会执行bin里的文件

npx webpack     // 不设置mode的情况下 打包出来的文件自动压缩

npx webpack --mode development  // 设置mode为开发模式，打包后的文件不被压缩
复制代码
```

当执行**npx webpack**命令的时候，webpack会自动查找项目中**src目录**下的**index.js**文件，然后进行打包，生成一个**dist目录**并存在一个打包好的**main.js**文件

这些算是0配置的操作了，名字都是定义好的，不能变，想想也很鸡肋

![img](https://user-gold-cdn.xitu.io/2018/4/25/162fd3991e6a27cf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2018/4/25/162fd39d67457424?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

webpack的使用还是在我们的配置方面，下面就进入我们的常规操作环节



## webpack是基于Node的

在项目下创建一个webpack.config.js(默认，可修改)文件来配置webpack

```
module.exports = {
    entry: '',               // 入口文件
    output: {},              // 出口文件
    module: {},              // 处理对应模块
    plugins: [],             // 对应的插件
    devServer: {},           // 开发服务器配置
    mode: 'development'      // 模式配置
}
复制代码
```

以上就是webpack的正常配置模块

★ 启动devServer需要安装一下webpack-dev-server

```
npm i webpack-dev-server -D
复制代码
```



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fd500b94644ca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

按照项目的结构，我们就从0开始去写一下配置吧



```
// webpack.config.js

const path = require('path');

module.exports = {
    entry: './src/index.js',    // 入口文件
    output: {
        filename: 'bundle.js',      // 打包后的文件名称
        path: path.resolve('dist')  // 打包后的目录，必须是绝对路径
    }
}
复制代码
```

上面就可以说是实现了最简单的webpack配置了，那接下来就打包一下看看

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300951712e1f7a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 配置执行文件

工作当中我们打包编译的时候一般都执行**npm run dev**这样的命令，既然是通过npm执行的命令，我们就应该找到package.json里的执行脚本去配置一下命令，这里如下图所示



![img](https://user-gold-cdn.xitu.io/2018/4/25/162fd57a53a0e108?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**npm run build**就是我们打包后的文件，这是生产环境下，上线需要的文件



**npm run dev**是我们开发环境下打包的文件，当然由于devServer帮我们把文件放到内存中了，所以并不会输出打包后的dist文件夹

通过npm run build之后会生成一个dist目录文件夹，就和上面打包后的样子一样了

## 多入口文件

多个入口可以有两种实现方式进行打包

- 一种是没有关系的但是要打包到一起去的，可以写一个数组，实现多个文件打包
- 另一种就是每一个文件都单独打包成一个文件的
- 下面就来看看这两种方式的写法

```
let path = require('path');

module.exports = {
    // 1.写成数组的方式就可以打出多入口文件，不过这里打包后的文件都合成了一个
    // entry: ['./src/index.js', './src/login.js'],
    // 2.真正实现多入口和多出口需要写成对象的方式
    entry: {
        index: './src/index.js',
        login: './src/login.js'
    },
    output: {
        // 1. filename: 'bundle.js',
        // 2. [name]就可以将出口文件名和入口文件名一一对应
        filename: '[name].js',      // 打包后会生成index.js和login.js文件
        path: path.resolve('dist')
    }
}
复制代码
```

这时候执行npm run build后，会生成打包好的两个js文件，如图所示

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300af9d7802153?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 配置Html模板

文件都打包好了，但是我们在使用的时候不能在dist目录下去创建一个html文件，然后去引用打包后的js吧，这不合理，实际开发中也不会这样

我们需要实现html打包功能，可以通过一个模板实现打包出引用好路径的html来

这就需要用到一个常用的插件了，**html-webpack-plugin**，用之前我们来安一下它

```
npm i html-webpack-plugin -D
复制代码
```

因为是个插件，所以需要在config.js里引用一下的

```
let path = require('path');
// 插件都是一个类，所以我们命名的时候尽量用大写开头
let HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        // 添加hash可以防止文件缓存，每次都会生成4位的hash串
        filename: 'bundle.[hash:4].js',   
        path: path.resolve('dist')
    },
    plugins: [
        // 通过new一下这个类来使用插件
        new HtmlWebpackPlugin({
            // 用哪个html作为模板
            // 在src目录下创建一个index.html页面当做模板来用
            template: './src/index.html',
            hash: true, // 会在打包好的bundle.js后面加上hash串
        })
    ]
}
复制代码
```

通过上面的配置后，我们再npm run build打包看一下现在是个什么样子了

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300c7f0dd28ade?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 多页面开发，怎么配置多页面

如果开发的时候不只一个页面，我们需要配置多页面，那么需要怎么来搞呢？不用担心，html-webpack-plugin插件自有办法，我们来观望一下

```
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    // 多页面开发，怎么配置多页面
    entry: {
        index: './src/index.js',
        login: './src/login.js'
    },
    // 出口文件  
    output: {                       
        filename: '[name].js',
        path: path.resolve('dist')
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',   
            filename: 'index.html',
            chunks: ['index']   // 对应关系,index.js对应的是index.html
        }),
        new HtmlWebpackPlugin({
            template: './src/login.html',
            filename: 'login.html',
            chunks: ['login']   // 对应关系,login.js对应的是login.html
        })
    ]
}
复制代码
```

继续npm run build看打包后的样子

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300d9f23bccd0e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

上面基本介绍完了html和js的打包配置了，现在我们还缺一个好兄弟css，webpack对css的解析需要用到loader，所以我们先提前安装好，待会好方便使用



## 引用CSS文件

可以在src/index.js里引入css文件，到时候直接打包到生产目录下

需要下载一些解析css样式的loader

```
npm i style-loader css-loader -D
// 引入less文件的话，也需要安装对应的loader
npm i less less-loader -D
复制代码
```

下面我们来看一下如何配置css文件的解析

```
// index.js
import './css/style.css';   // 引入css
import './less/style.less'; // 引入less

console.log('这里是打包文件入口-index.js');

// webpack.config.js
module.exports = {
    entry: {
        index: './src/index.js'
    },
    output: {
        filename: 'bundle.js',
        path: path.resolve('dist')
    },
    module: {
        rules: [
            {
                test: /\.css$/,     // 解析css
                use: ['style-loader', 'css-loader'] // 从右向左解析
                /* 
                    也可以这样写，这种方式方便写一些配置参数
                    use: [
                        {loader: 'style-loader'},
                        {loader: 'css-loader'}
                    ]
                */
            }
        ]
    }
}
复制代码
```



![img](https://user-gold-cdn.xitu.io/2018/4/26/16300f20c7d6587c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 此时打包后的css文件是以行内样式style的标签写进打包后的html页面中，如果样式很多的话，我们更希望直接用link的方式引入进去，这时候需要把**css拆分出来**
- **extract-text-webpack-plugin**插件相信用过的人都知道它是干什么的，它的功效就在于会将打包到js里的css文件进行一个拆分

### 拆分CSS

```
// @next表示可以支持webpack4版本的插件
npm i extract-text-webpack-plugin@next -D
复制代码
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');
// 拆分css样式的插件
let ExtractTextWebpackPlugin = require('extract-text-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        filaneme: 'bundle.js',
        path: path.resolve('dist')
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextWebpackPlugin.extract({
                    // 将css用link的方式引入就不再需要style-loader了
                    use: 'css-loader'       
                })
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
        }),
        // 拆分后会把css文件放到dist目录下的css/style.css
        new ExtractTextWebpackPlugin('css/style.css')  
    ]
}
复制代码
```

此时拆分完css后，打包的html页面就以link的方式去引入css了，这样很好

![img](https://user-gold-cdn.xitu.io/2018/4/26/16300fa1d7d47ebd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

当然大家很多都说另外一个插件也是可以办到的，那就是mini-css-extract-plugin，是的可以说它是为webpack4而生的，而之所以上来就没有介绍是因为还不成熟，还有很多bug需要去解决的



不过既然大家都知道它，那就顺便也提一下吧

```
npm i mini-css-extract-plugin -D
复制代码
```

使用起来和上面的插件是差不多的

```
let MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader, 'css-loader']
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/a.css'   // 指定打包后的css
        })
    ]
}
复制代码
```

### 拆分成多个css

这里要着重说一下上面两个插件的区别了，我个人还是建议用extract-text-webpack-plugin的，毕竟从之前的版本承接下来的，虽然在安包的时候需要@next，但是还是值得信赖的

而且现在的extract-text-webpack-plugin也支持了拆分成多个css，而目前mini-css-extract-plugin还不支持此功能

```
// 正常写入的less
let styleLess = new ExtractTextWebpackPlugin('css/style.css');
// reset
let resetCss = new ExtractTextWebpackPlugin('css/reset.css');

module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: resetCss.extract({
                    use: 'css-loader'
                })
            },
            {
                test: /\.less$/,
                use: styleLess.extract({
                    use: 'css-loader'
                })
            }
        ]
    },
    plugins: [
        styleLess,
        resetCss
    ]
}
复制代码
```

通过这样操作后可以打包成两个不同的css文件，如下图

<img src="https://user-gold-cdn.xitu.io/2018/4/26/16300fa1d7d47ebd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" alt="img" style="zoom:50%;" />



## 引用图片

- 处理图片方面，也需要loader

```
npm i file-loader url-loader -D
复制代码
```

如果是在css文件里引入的如背景图之类的图片，就需要指定一下相对路径

```
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ExtractTextWebpackPlugin.extract({
                    use: 'css-loader',
                    publicPath: '../'
                })
            },
            {
                test: /\.(jpe?g|png|gif)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 8192,    // 小于8k的图片自动转成base64格式，并且不会存在实体图片
                            outputPath: 'images/'   // 图片打包后存放的目录
                        }
                    }
                ]
            }
        ]
    }
}
复制代码
```

在css中指定了publicPath路径这样就可以根据相对路径引用到图片资源了，如下图所示

![img](https://user-gold-cdn.xitu.io/2018/4/26/163015e474789835?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 页面img引用图片

页面中经常会用到img标签，img引用的图片地址也需要一个loader来帮我们处理好

```
npm i html-withimg-loader -D
复制代码
module.exports = {
    module: {
        rules: [
            {
                test: /\.(htm|html)$/,
                use: 'html-withimg-loader'
            }
        ]
    }
}
复制代码
```

这样再打包后的html文件下img就可以正常引用图片路径了

![img](https://user-gold-cdn.xitu.io/2018/4/26/1630161fbaa1d225?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 引用字体图片和svg图片

字体图标和svg图片都可以通过file-loader来解析

```
module.exports = {
    module: {
        rules: [
            {
                test: /\.(eot|ttf|woff|svg)$/,
                use: 'file-loader'
            }
        ]
    }
}
复制代码
```

这样即使样式中引入了这类格式的图标或者图片都没有问题了，img如果也引用svg格式的话，配合上面写好的html-withimg-loader就都没有问题了

## 添加CSS3前缀

通过postcss中的autoprefixer可以实现将CSS3中的一些需要兼容写法的属性添加响应的前缀，这样省去我们不少的时间

由于也是一个loader加载器，我们也需要先安装一下

```
npm i postcss-loader autoprefixer -D
复制代码
```

安装后，我们还需要像webpack一样写一个config的配置文件，在项目根目录下创建一个**postcss.config.js**文件，配置如下：

```
module.exports = {
    plugins: [require('autoprefixer')]  // 引用该插件即可了
}
复制代码
```

然后在webpack里配置postcss-loader

```
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader', 'postcss-loader']
            }
        ]
    }
}
复制代码
```

## 转义ES6

在实际开发中，我们在大量的使用着ES6及之后的api去写代码，这样会提高我们写代码的速度，不过由于低版本浏览器的存在，不得不需要转换成兼容的代码，于是就有了常用的Babel了

Babel会将ES6的代码转成ES5的代码

那么不再多说，既然要使用它，就先来安一下

```
npm i @babel/core babel-loader @babel/preset-env -D
复制代码
```

当把这些都安好后，我们就开始配置，通过webpack里配置的babel-loader，就可以做到代码转成ES5了

```
module.exports = {
    module: {
        rules: [
            {
                test:/\.js$/,
                use: {
                    loader: 'babel-loader',
                    // 配置选项里的presets
                    // 包含ES6还有之后的版本和那些仅仅是草案的内容
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
                include: /src/,          // 只转化src目录下的js
                exclude: /node_modules/  // 排除掉node_modules，优化打包速度
            }
        ]
    }
}
复制代码
```

## 清空打包文件夹

在我们每次npm run build的时候都会在dist目录下创建很多打好的包，如果积累过多可能也会混乱

所以应该在每次打包之前将dist目录下的文件都清空，然后再把打好包的文件放进去

这里提供一个clean-webpack-plugin插件

```
npm i clean-webpack-plugin -D
复制代码
let CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
    plugins: [
        // 打包前先清空
        new CleanWebpackPlugin('dist')  
    ]
}
复制代码
```

## 启动静态服务器

启动一个静态服务器，默认会**自动刷新**，就是说你对html,css,js文件做了修改并保存后，浏览器会默认刷新一次展现修改后的效果

正常情况下我们都是在开发环境中开发项目，所以之前配置的脚本"dev"可以派上用场了，在执行**npm run dev**命令后，会启动静态服务器，我们访问**localhost:3000**端口就可以看到开发的页面内容了

如果devServer里open设为true后，会自动打开浏览器

```
module.exports = {
    devServer: {
        contentBase: './dist',
        host: 'localhost',      // 默认是localhost
        port: 3000,             // 端口
        open: true,             // 自动打开浏览器
        hot: true               // 开启热更新
    }
}
复制代码
```

当然在npm run dev命令下，打包的文件存在于内存中，并不会产生在dist目录下

### 热更新和自动刷新的区别

在配置devServer的时候，如果hot为true，就代表开启了热更新

But这并没那么简单，因为热更新还需要配置一个webpack自带的插件并且还要在主要js文件里检查是否有module.hot

下面就让我们直接看下代码是如何实现的

```
// webpack.config.js
let webpack = require('webpack');

module.exports = {
    plugins: [
        // 热更新，热更新不是刷新
        new webpack.HotModuleReplacementPlugin()
    ],
    devServer: {
        contentBase: './dist',
        hot: true,
        port: 3000
    }
}

// 此时还没完虽然配置了插件和开启了热更新，但实际上并不会生效

// index.js
let a = 'hello world';
document.body.innerHTML = a;
console.log('这是webpack打包的入口文件');

// 还需要在主要的js文件里写入下面这段代码
if (module.hot) {
    // 实现热更新
    module.hot.accept();
}
复制代码
```

以上index.js中的内容，如果将变量a的值进行修改保存后，会在不刷新页面的情况下直接修改掉，这样就实现了热更新

那么热更新从现在看来和自动刷新浏览器的区别也不是太大嘛！自动刷新也是可以接受的啊

其实不然，热更新的好处可能在vue或者react中有更大的发挥，其中某一个组件被修改的时候就会针对这个组件进行热更新了，这里用到vue或react的同学去实际体验一下吧

## resolve解析

在webpack的配置中，resolve我们常用来配置别名和省略后缀名

```
module.exports = {
    resolve: {
        // 别名
        alias: {
            $: './src/jquery.js'
        },
        // 省略后缀
        extensions: ['.js', '.json', '.css']
    },
}
复制代码
```

这个配置在webpack中比较简单，我们也就不再叙述了，下面来看点干货

## 提取公共代码

在webpack4之前，提取公共代码都是通过一个叫CommonsChunkPlugin的插件来办到的。到了4以后，内置了一个一模一样的功能，而且起了一个好听的名字叫“优化”

下面我们就来看看如何提取公共代码

```
// 假设a.js和b.js都同时引入了jquery.js和一个写好的utils.js
// a.js和b.js
import $ from 'jquery';
import {sum} from 'utils';
复制代码
```

那么他们两个js中其中公共部分的代码就是jquery和utils里的代码了

可以针对第三方插件和写好的公共文件

```
module.exports = {
    entry: {
        a: './src/a.js',
        b: './src/b.js'
    },
    output: {
        filename: '[name].js',
        path: path.resolve('dust')
    },
    // 提取公共代码
+   optimization: {
        splitChunks: {
            cacheGroups: {
                vendor: {   // 抽离第三方插件
                    test: /node_modules/,   // 指定是node_modules下的第三方包
                    chunks: 'initial',
                    name: 'vendor',  // 打包后的文件名，任意命名    
                    // 设置优先级，防止和自定义的公共代码提取时被覆盖，不进行打包
                    priority: 10    
                },
                utils: { // 抽离自己写的公共代码，utils这个名字可以随意起
                    chunks: 'initial',
                    name: 'utils',  // 任意命名
                    minSize: 0    // 只要超出0字节就生成一个新包
                }
            }
        }
+   },
    plugins: [
        new HtmlWebpackPlugin({
            filename: 'a.html',
            template: './src/index.html',  // 以index.html为模板
+           chunks: ['vendor', 'a']
        }),
        new HtmlWebpackPlugin({
            filename: 'b.html',
            template: './src/index.html',  // 以index.html为模板
+           chunks: ['vendor', 'b']
        })
    ]
}
复制代码
```

通过以上配置，可以把引入到a.js和b.js中的这部分公共代码提取出来，如下图所示

![img](https://user-gold-cdn.xitu.io/2018/4/27/16302c4bc8dd3596?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 指定webpack配置文件

在package.json的脚步里，我们可以配置调用不同的webpack配置文件进行打包，举个栗子



![img](https://user-gold-cdn.xitu.io/2018/4/27/16302e1dd30d1f06?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这样的好处在于可以针对不同的需求进行一个特定的打包配置



作者：chenhongdong
链接：https://juejin.im/post/6844903599080734728


