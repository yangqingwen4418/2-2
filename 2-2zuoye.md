# 一、简答题

#### 1、Webpack 的构建流程主要有哪些环节？如果可以请尽可能详尽的描述 Webpack 打包的整个过程。

答：流程有：1、执行yarn init --yes命令初始化一个package.json文件，

​						2、安装webpack：yarn add webpack webpack-cli --dev，

​						3、执行命令yarn/npx webpack --version检查下webpack安装成功与否，

​						4、创建配置文件webpack.config.js，自定义配置；

​						5、运用webpack内置的各种特性、loader和plugin完成各种任务；

​						6、执行打包命令yarn/npx webpack，也可以把打包命令webpack写到package.json中的“scripts”中，注意development、none、production模式的区别；

​						7、直接在HTML页面引入 webpack 最终生成的dist目录的页面脚本即可。



#### 2、Loader 和 Plugin 有哪些不同？请描述一下开发 Loader 和 Plugin 的思路。

答：一、从功能作用的角度区分：
1、loader： loader从字面的意思理解，是 加载 的意思。由于webpack 本身只能打包commonjs规范的js文件，所以，针对css，图片等格式的文件没法打包，就需要引入第三方的模块进行打包。  loader虽然是扩展了 webpack ，但是它只专注于转化文件（transform）这一个领域，完成压缩，打包，语言翻译。 loader是运行在NodeJS中。 仅仅只是为了打包。如：css-loader和style-loader模块是为了打包css的，  babel-loader和babel-core模块时为了把ES6的代码转成ES5， url-loader和file-loader是把图片进行打包的。       

2、plugin完成的是loader不能完成的功能，plugin扩展了webpack的功能，但是 plugin 是作用于webpack本身上的。而且plugin不仅只局限在打包，资源的加载上，它的功能要更加丰富。从打包优化和压缩，到重新定义环境变量，功能强大到可以用来处理各种各样的任务。webpack提供了很多开箱即用的插件：CommonChunkPlugin主要用于提取第三方库和公共模块，避免首屏加载的bundle文件，或者按需加载的bundle文件体积过大，导致加载时间过长，是一把优化的利器。而在多页面应用中，更是能够为每个页面间的应用程序共享代码创建bundle。  插件可以携带参数，所以在plugins属性传入new实例。如：

1）、针对html文件打包和拷贝（还有很多设置）的插件：html-webpack-plugin。不但完成了html文件的拷贝，打包，还给html中自动增加了引入打包后的js文件的代码（<script src=""></script>），还能指明把js文件引入到html文件的底部等等。

二、从运行时机的角度区分


 1 . loader运行在打包文件之前（loader为在模块加载时的预处理文件）
 2. plugins在整个编译周期都起作用。

    

开发loader思路：

由于Webpack是运行在Node.js之上的，一个`Loader`其实就是一个Node.js模块，这个模块需要导出一个函数。 这个导出的函数的工作就是获得处理前的原内容，对原内容执行处理后，返回处理后的内容。由于`Loader`运行在Node.js中，你可以调用任何Node.js自带的API，或者安装第三方模块进行调用，Webpack还提供一些API供`Loader`调用。一个`Loader`的职责是单一的，只需要完成一种转换。 如果一个源文件需要经历多步转换才能正常使用，就通过多个`Loader`去转换。 在调用多个`Loader`去转换一个文件时，每个`Loader`会链式的顺序执行， 第一个Loader将会拿到需处理的原内容，上一个`Loader`处理后的结果会传给下一个接着处理，最后的`Loader`将处理后的最终结果返回给Webpack。
 所以，在你开发一个`Loader`时，请保持其职责的单一性，你只需关心输入和输出

`Loader`有同步和异步之分，上面介绍的`Loader`都是同步的`Loader`，因为它们的转换流程都是同步的，转换完成后再返回结果。 但在有些场景下转换的步骤只能是异步完成的，例如你需要通过网络请求才能得出结果，如果采用同步的方式网络请求就会阻塞整个构建，导致构建非常缓慢。
 在转换步骤是异步时，你可以这样：

module.exports = function(source) {
    // 告诉 Webpack 本次转换是异步的，Loader 会在 callback 中回调结果
    var callback = this.async();
    someAsyncOperation(source, function(err, result, sourceMaps, ast) {
        // 通过 callback 返回异步执行后的结果
        callback(err, result, sourceMaps, ast);
    });
};

在有些情况下，有些转换操作需要大量计算非常耗时，如果每次构建都重新执行重复的转换操作，构建将会变得非常缓慢。为此，Webpack会默认缓存所有`Loader`的处理结果，也就是说在需要被处理的文件或者其依赖的文件没有发生变化时， 是不会重新调用对应的`Loader`去执行转换操作的。
 如果想让Webpack不缓存该`Loader`的处理结果，可以这样：

module.exports = function(source) {
  // 关闭该 Loader 的缓存功能
  this.cacheable(false);
  return source;
};

##### 获得Loader的options

要在自己编写的Loader中获取到用户传入的`options`，需要这样做：

const loaderUtils = require('loader-utils');
module.exports = function(source) {
  // 获取到用户给当前 Loader 传入的 options
  const options = loaderUtils.getOptions(this);
  return source;
};

##### 处理二进制数据

在默认的情况下，Webpack传给`Loader`的原内容都是UTF-8格式编码的字符串。 但有些场景下`Loader`不是处理文本文件，而是处理二进制文件，例如`file-loader`，就需要Webpack给`Loader`传入二进制格式的数据。 为此，你需要这样编写`Loader`：

module.exports = function(source) {
    // 在 exports.raw === true 时，Webpack 传给 Loader 的 source 是 Buffer 类型的
    source instanceof Buffer === true;
    // Loader 返回的类型也可以是 Buffer 类型的
    // 在 exports.raw !== true 时，Loader 也可以返回 Buffer 类型的结果
    return source;
};
// 通过 exports.raw 属性告诉 Webpack 该 Loader 是否需要二进制数据 
module.exports.raw = true;

以上代码中最关键的代码是最后一行`module.exports.raw = true;`，没有该行`Loader`只能拿到字符串。

#### 加载本地Loader

在开发`Loader`的过程中，为了测试编写的`Loader`是否能正常工作，需要把它配置到Webpack中后，才可能会调用该`Loader`。使用的`Loader`都是通过Npm安装的，要使用`Loader`时会直接使用`Loader`的名称，代码如下：

module.exports = {
  module: {
    rules: [
      {
        test: /\.css/,
        use: ['style-loader'],
      },
    ]
  },
};

如果还采取以上的方法去使用本地开发的`Loader`将会很麻烦，因为你需要确保编写的`Loader`的源码是在`node_modules`目录下。 为此你需要先把编写的`Loader`发布到Npm仓库后再安装到本地项目使用。
 解决以上问题的便捷方法有两种，分别如下：

##### Npm link

Npm link专门用于开发和调试本地Npm模块，能做到在不发布模块的情况下，把本地的一个正在开发的模块的源码链接到项目的`node_modules`目录下，让项目可以直接使用本地的Npm模块。 由于是通过软链接的方式实现的，编辑了本地的Npm模块代码，在项目中也能使用到编辑后的代码。
 完成Npm link的步骤如下：

1. 确保正在开发的本地Npm模块（也就是正在开发的Loader）的`package.json`已经正确配置好；
2. 在本地Npm模块根目录下执行`npm link`，把本地模块注册到全局；
3. 在项目根目录下执行`npm link loader-name`，把第2步注册到全局的本地Npm模块链接到项目的`node_moduels`下，其中的`loader-name`是指在第1步中的`package.json`文件中配置的模块名称。

链接好`Loader`到项目后你就可以像使用一个真正的 Npm 模块一样使用本地的`Loader`了。

##### ResolveLoader

`ResolveLoader`用于配置Webpack如何寻找`Loader`。 默认情况下只会去`node_modules`目录下寻找，为了让Webpack加载放在本地项目中的`Loader`需要修改`resolveLoader.modules`。
 假如本地的`Loader`在项目目录中的`./loaders/loader-name`中，则需要如下配置：

module.exports = {
  resolveLoader:{
    // 去哪些目录下寻找 Loader，有先后顺序之分
    modules: ['node_modules','./loaders/'],
  }
}

加上以上配置后，Webpack会先去`node_modules`项目下寻找`Loader`，如果找不到，会再去`./loaders/`目录下寻找

#### 其它Loader API

除了以上在`Loader`中能调用的Webpack API外，还存在以下常用API：

- `this.context`：当前处理文件的所在目录，假如当前`Loader`处理的文件是`/src/main.js`，则`this.context`就等于`/src`。
- `this.resource`：当前处理文件的完整请求路径，包括`querystring`，例如`/src/main.js?name=1`。
- `this.resourcePath`：当前处理文件的路径，例如`/src/main.js`。
- `this.resourceQuery`：当前处理文件的`querystring`。
- `this.target`：等于Webpack配置中的`Target`。
- `this.loadModule`：当`Loader`在处理一个文件时，如果依赖其它文件的处理结果才能得出当前文件的结果时， 就可以通过`this.loadModule(request: string, callback: function(err, source, sourceMap, module))`去获得`request`对应文件的处理结果。
- `this.resolve`：像`require`语句一样获得指定文件的完整路径，使用方法为`resolve(context: string, request: string, callback: function(err, result: string))`。
- `this.addDependency`：给当前处理文件添加其依赖的文件，以便再其依赖的文件发生变化时，会重新调用`Loader`处理该文件。使用方法为`addDependency(file: string)`。
- `this.addContextDependency`：和`addDependency`类似，但`addContextDependency`是把整个目录加入到当前正在处理文件的依赖中。使用方法为`addContextDependency(directory: string)`。
- `this.clearDependencies`：清除当前正在处理文件的所有依赖，使用方法为`clearDependencies()`。
- `this.emitFile`：输出一个文件，使用方法为`emitFile(name: string, content: Buffer|string, sourceMap: {...})`。



开发plugin思路：

`plugin` 通常是在 `webpack` 在打包的某个时间节点做一些操作，我们使用 `plugin` 的时候，一般都是 `new Plugin()` 这种形式使用，所以，首先应该明确的是， `plugin` 应该是一个类。

Webpack通过`Plugin`机制让其更加灵活，以适应各种应用场景。 在Webpack运行的生命周期中会广播出许多事件，`Plugin`可以监听这些事件，在合适的时机通过Webpack提供的API改变输出结果。
 一个最基础的`Plugin`的代码是这样的：

```jsx
class BasicPlugin{
  // 在构造函数中获取用户给该插件传入的配置
  constructor(options){
  }

  // Webpack 会调用 BasicPlugin 实例的 apply 方法给插件实例传入 compiler 对象
  apply(compiler){
    compiler.plugin('compilation',function(compilation) {
    })
  }
}
// 导出 Plugin
module.exports = BasicPlugin;
```

在使用这个`Plugin`时，相关配置代码如下：

```tsx
const BasicPlugin = require('./BasicPlugin.js');
module.export = {
  plugins:[
    new BasicPlugin(options),
  ]
}
```

Webpack启动后，在读取配置的过程中会先执行`new BasicPlugin(options)`初始化一个`BasicPlugin`获得其实例。 在初始化`compiler`对象后，再调用`basicPlugin.apply(compiler)`给插件实例传入`compiler`对象。 插件实例在获取到`compiler`对象后，就可以通过`compiler.plugin(事件名称, 回调函数)`监听到Webpack广播出来的事件。 并且可以通过`compiler`对象去操作Webpack。

#### Compiler和Compilation

在开发`Plugin`时最常用的两个对象就是`Compiler`和`Compilation`，它们是`Plugin`和Webpack之间的桥梁。`Compiler`和`Compilation`的含义如下：

- `Compiler`对象包含了Webpack环境所有的的配置信息，包含`options`，`loaders`，`plugins`这些信息，这个对象在Webpack启动时候被实例化，它是全局唯一的，可以简单地把它理解为Webpack实例；
- `Compilation`对象包含了当前的模块资源、编译生成资源、变化的文件等。当Webpack以开发模式运行时，每当检测到一个文件变化，一次新的`Compilation`将被创建。`Compilation`对象也提供了很多事件回调供插件做扩展。通过`Compilation`也能读取到`Compiler`对象。

`Compiler`和`Compilation`的区别在于：`Compiler`代表了整个Webpack从启动到关闭的生命周期，而`Compilation`只是代表了一次新的编译。

#### 事件流

Webpack就像一条生产线，要经过一系列处理流程后才能将源文件转换成输出结果。 这条生产线上的每个处理流程的职责都是单一的，多个流程之间有存在依赖关系，只有完成当前处理后才能交给下一个流程去处理。 插件就像是一个插入到生产线中的一个功能，在特定的时机对生产线上的资源做处理。
 Webpack通过Tapable来组织这条复杂的生产线。 Webpack在运行过程中会广播事件，插件只需要监听它所关心的事件，就能加入到这条生产线中，去改变生产线的运作。 Webpack的事件流机制保证了插件的有序性，使得整个系统扩展性很好。
 Webpack的事件流机制应用了观察者模式，和Node.js中的`EventEmitter`非常相似。`Compiler`和`Compilation`都继承自`Tapable`，可以直接在`Compiler`和`Compilation`对象上广播和监听事件，方法如下：

```
/**
* 广播出事件
* event-name 为事件名称，注意不要和现有的事件重名
* params 为附带的参数
*/
compiler.apply('event-name',params);
/**
* 监听名称为 event-name 的事件，当 event-name 事件发生时，函数就会被执行。
* 同时函数中的 params 参数为广播事件时附带的参数。
*/
compiler.plugin('event-name',function(params) {

});
```

同理，`compilation.apply`和`compilation.plugin`使用方法和上面一致。
 在开发插件时，你可能会不知道该如何下手，因为你不知道该监听哪个事件才能完成任务。
 在开发插件时，还需要注意以下两点：

- 只要能拿到`Compiler`或`Compilation`对象，就能广播出新的事件，所以在新开发的插件中也能广播出事件，给其它插件监听使用。
- 传给每个插件的`Compiler`和`Compilation`对象都是同一个引用。也就是说在一个插件中修改了`Compiler`或`Compilation`对象上的属性，会影响到后面的插件。
- 有些事件是异步的，这些异步的事件会附带两个参数，第二个参数为回调函数，在插件处理完任务时需要调用回调函数通知Webpack，才会进入下一处理流程。例如：

```jsx
compiler.plugin('emit',function(compilation, callback) {
  // 支持处理逻辑
  // 处理完毕后执行 callback 以通知 Webpack 
  // 如果不执行 callback，运行流程将会一直卡在这不往下执行 
  callback();
});
```

#### 常用API

插件可以用来修改输出文件、增加输出文件、甚至可以提升Webpack性能、等等，总之插件通过调用 Webpack提供的API能完成很多事情。 由于Webpack提供的API非常多，有很多API很少用的上，又加上篇幅有限，下面来介绍一些常用的API。

#### 读取输出资源、代码块、模块及其依赖

有些插件可能需要读取Webpack的处理结果，例如输出资源、代码块、模块及其依赖，以便做下一步处理。
 在`emit`事件发生时，代表源文件的转换和组装已经完成，在这里可以读取到最终将输出的资源、代码块、模块及其依赖，并且可以修改输出资源的内容。 插件代码如下：

```php
class Plugin {
  apply(compiler) {
    compiler.plugin('emit', function (compilation, callback) {
      // compilation.chunks 存放所有代码块，是一个数组
      compilation.chunks.forEach(function (chunk) {
        // chunk 代表一个代码块
        // 代码块由多个模块组成，通过 chunk.forEachModule 能读取组成代码块的每个模块
        chunk.forEachModule(function (module) {
          // module 代表一个模块
          // module.fileDependencies 存放当前模块的所有依赖的文件路径，是一个数组
          module.fileDependencies.forEach(function (filepath) {
          });
        });

        // Webpack 会根据 Chunk 去生成输出的文件资源，每个 Chunk 都对应一个及其以上的输出文件
        // 例如在 Chunk 中包含了 CSS 模块并且使用了 ExtractTextPlugin 时，
        // 该 Chunk 就会生成 .js 和 .css 两个文件
        chunk.files.forEach(function (filename) {
          // compilation.assets 存放当前所有即将输出的资源
          // 调用一个输出资源的 source() 方法能获取到输出资源的内容
          let source = compilation.assets[filename].source();
        });
      });

      // 这是一个异步事件，要记得调用 callback 通知 Webpack 本次事件监听处理结束。
      // 如果忘记了调用 callback，Webpack 将一直卡在这里而不会往后执行。
      callback();
    })
  }
}
```

#### 监听文件变化

Webpack会从配置的入口模块出发，依次找出所有的依赖模块，当入口模块或者其依赖的模块发生变化时， 就会触发一次新的`Compilation`。
 在开发插件时经常需要知道是哪个文件发生变化导致了新的`Compilation`，为此可以使用如下代码：

```tsx
// 当依赖的文件发生变化时会触发 watch-run 事件
compiler.plugin('watch-run', (watching, callback) => {
    // 获取发生变化的文件列表
    const changedFiles = watching.compiler.watchFileSystem.watcher.mtimes;
    // changedFiles 格式为键值对，键为发生变化的文件路径。
    if (changedFiles[filePath] !== undefined) {
      // filePath 对应的文件发生了变化
    }
    callback();
});
```

默认情况下Webpack只会监视入口和其依赖的模块是否发生变化，在有些情况下项目可能需要引入新的文件，例如引入一个HTML文件。 由于 JavaScript 文件不会去导入HTML文件，Webpack就不会监听HTML文件的变化，编辑HTML文件时就不会重新触发新的`Compilation`。 为了监听HTML文件的变化，我们需要把HTML文件加入到依赖列表中，为此可以使用如下代码：

```tsx
compiler.plugin('after-compile', (compilation, callback) => {
  // 把 HTML 文件添加到文件依赖列表，好让 Webpack 去监听 HTML 模块文件，在 HTML 模版文件发生变化时重新启动一次编译
    compilation.fileDependencies.push(filePath);
    callback();
});
```

#### 修改输出资源

有些场景下插件需要修改、增加、删除输出的资源，要做到这点需要监听`emit`事件，因为发生`emit`事件时所有模块的转换和代码块对应的文件已经生成好， 需要输出的资源即将输出，因此`emit`事件是修改Webpack输出资源的最后时机。
 所有需要输出的资源会存放在`compilation.assets`中，`compilation.assets`是一个键值对，键为需要输出的文件名称，值为文件对应的内容。
 设置`compilation.assets`的代码如下：

```tsx
compiler.plugin('emit', (compilation, callback) => {
  // 设置名称为 fileName 的输出资源
  compilation.assets[fileName] = {
    // 返回文件内容
    source: () => {
      // fileContent 既可以是代表文本文件的字符串，也可以是代表二进制文件的 Buffer
      return fileContent;
      },
    // 返回文件大小
      size: () => {
      return Buffer.byteLength(fileContent, 'utf8');
    }
  };
  callback();
});
```

读取`compilation.assets`的代码如下：

```tsx
compiler.plugin('emit', (compilation, callback) => {
  // 读取名称为 fileName 的输出资源
  const asset = compilation.assets[fileName];
  // 获取输出资源的内容
  asset.source();
  // 获取输出资源的文件大小
  asset.size();
  callback();
});
```

#### 判断Webpack使用了哪些插件

在开发一个插件时可能需要根据当前配置是否使用了其它某个插件而做下一步决定，因此需要读取Webpack当前的插件配置情况。 以判断当前是否使用了`ExtractTextPlugin`为例，可以使用如下代码：

```jsx
// 判断当前配置使用使用了 ExtractTextPlugin，
// compiler 参数即为 Webpack 在 apply(compiler) 中传入的参数
function hasExtractTextPlugin(compiler) {
  // 当前配置所有使用的插件列表
  const plugins = compiler.options.plugins;
  // 去 plugins 中寻找有没有 ExtractTextPlugin 的实例
  return plugins.find(plugin=>plugin.__proto__.constructor === ExtractTextPlugin) != null;
}
```

也可以参考该链接：https://www.jb51.net/article/186020.htm



# 二、编程题

## 1、使用 Webpack 实现 Vue 项目打包任务

 我的解说：

**首先—思路分析**。

​		拿到项目明确需求，该项目的需求为主要把src目录下的各类文件采用webpack、loader、plugin、eslint等知识，根据不同的mode实现不同模式下的集成打包任务。

​		明确了目标和需求后，接下来分析项目目录结构，如图一，src下面有.png、.vue、.less、

.js、.html等格式文件。

![](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707094531659.png)

​																	图一：src目录结构分析

​		html文件因为在public目录下有一个index.html的模板，所以我考虑会用html-webpack-plugin的插件在打包目录dist下自动生成index.html文件。

​		vue格式文件，采用vue-loade、vue-loader-plugin和vue-template-compiler。

​		png格式文件，看了2张图片均低于10KB大小，故采用url-loader，把图片形成base64码嵌入代码中，减少请求。当然也可以采用file-loader，但存在请求增加，性能不够优化问题。

​		less格式文件采用less-loader、css-loader、style-loader或mini-css-extract-plugin 插件和optimize-css-assets-webpack-plugin插件压缩css文件。

​		js格式文件采用webpack默认的内置功能配合babel-loader、terser-webpack-plugin压缩js文件的插件。

​		安装eslint、eslint-loader、stylelint、prettier等插件，使用eslint功能。

​		针对不同的mode，在项目根目录下分别建立webpack.prod.js、webpack.dev.js和webpack.common.js，然后安装webpack-merge合并共同的模块。

**其次**，**做一些前期的准备操作：**

​		1、执行npm install安装相应的依赖；

​		2、然后安装webpack：npm install webpack --save-dev和webpack-cli：npm install webpack-cli -g，这里，我之所以分开装是因为，我之前用npm install webpack webpack-cli命令时，发现报错，后面到网上查说webpack4版本把webpack和webpack-cli分开了，单独把webpack-cli分开全局装，就不会报错找不到webpack或webpack-cli的错误。

​		3、以--save-dev本地安装安装上面提到的所有插件。

**最后，具体操作：**

​		1、在项目根目录中新建webpack.prod.js、webpack.dev.js和webpack.common.js，针对各种production和development模式的需求做相应配置，（注意，production模式下的特性和打包上线前才需要的动作，如copywebpackplugin，得配置到webpack.prod.js中。），同时用webpack-merge插件把webpack.common.js中的两种模式的共同配置引入进来。然后把共同的配置如处理js文件写入webpack.common.js中，具体看下图或项目文件。最后注意到package.json中“scripts”中配置指令。

"scripts": {

​    "serve": "webpack-dev-server --inline --progress --config webpack.dev.js",

​    "build": "webpack --inline --progress --config webpack.prod.js",

  },

然后分别执行npm run-script build/serve

webpack.common.js内容：

![image-20200707155919743](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707155919743.png)

![image-20200707155945611](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707155945611.png)

webpac.dev.js内容：

![image-20200707155556216](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707155556216.png)![image-20200707155716288](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707155716288.png)

![image-20200707155741447](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707155741447.png)

webpack.prod.js内容：

![image-20200707160104545](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707160104545.png)

![image-20200707160124579](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707160124579.png)

![image-20200707160135856](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200707160135856.png)

​       2、配置eslint步骤：

​		在终端输入***\*npx eslint -–init\****，第一个问题选择最后一个最长的检查代码语法、问题代码及风格第二个问题语法风格，ESModule、commonjs和none，第三个选择有无框架，第四个选择ts，第五个代码运行在什么环境（浏览器或node），第六个选择怎样的代码风格，选市场上主流风格，standard不需要分号，第七个问题配置文件以何种格式存放，选js，最后一个问题，安装额外插件，选yes，npm会自动安装。一切就绪后，项目根目录下会多出.eslintrc.js的eslint配置文件。

​		然后，webpack是通过loader集成eslint的，非插件。一定要注意eslint-loader是放到js的loader的最后，率先执行。也可以为js文件重新添加***\*loader，{test：/\.js$/,exclude:/node_module/,use:’eslint-loader’,enforce:’pre’\****},然后执行npx webpack即可自动检查代码中的问题。

​	    3、执行打包命令，production模式下执行npm run-script serve，development模式下执行npm run-script serve命令。

