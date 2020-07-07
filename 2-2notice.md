2、动态生成html，先安装插件***\*yarn add html-webpack-plugin --dev\****，后在配置文件导入,注意的是与CleanWebpackPlugin不同，该插件默认导出就是一个插件类型，不需要解构成员。故***\*const HtmlWebpackPlugin = require(‘html-webpack-plugin’)\****，后在***\*plugins:[new HtmlWebpackPlugin({title：‘webpack’，meta：{viewport：‘width=device-width’}，template：‘./src/index.html’})]\****，在dist目录自动生成index.html，注意index.html默认script引入的脚本位置为dist/bundle.js，这是因为之前设置了publiPath：dist/,注释掉即可为bundle.js，如果小量html配置，可在 new HtmlWebpackPlugin()中加入对象配置{}，大量则需要html配置，则需要html模板，需要动态输入的内容采用lodash的模板语法***\*<% HtmlWebpackPlugin.options.title %>\****输出，options属性可以访问到插件的配置数据.同时输出多个页面文件，第一个new HtmlWebpackPlugin()创建的是index.html,在创建一个new HtmlWebpackPlugin({filename：‘about.html’})实例，则会创建about.html。总结一个new HtmlWebpackPlugin()创建一个实例，多个new HtmlWebpackPlugin()则创建多个实例html。

3、资源文件复制用***\*yarn add copy-webpack-plugin\****，配置文件开头导入后，在***\*plugins：{new CopyWebpackPlugin({‘public(/\*\*)’})}\****。总结，需要去github查看官网用法，可以在github.com上搜索其他plugin，如imagemin webpack plugin。

4、相比loader，plugin拥有更宽的能力范围，loader只在加载模块的时候才工作，而插件的工作范围几乎可以触碰到webpack每一个环节，plugin通过钩子机制实现，类似web中的事件，为便于工作，webpack几乎给每个环节埋下了钩子，可以不同的环节挂载不同的人物，官方api文档可查具体定义了那些钩子，webpack要求插件必须是一个函数或是一个包含apply方法的对象，一般把插件定义成一个对象，在这个对象上定义一个apply方法，这个方法会在webpack启动时被自动调用，通过这个类型构建一个实例，然后去使用。eg：class MyPlugin {apply(compiler){compiler.hooks.emit.tap(‘MyPlugin’，compilation=>{for(const name in compilation.assets){console.log(compilation.

assets[name].source())}})}},然后再plugins：{new MyPlugin()}

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps53D5.tmp.jpg) 

compiler是webpack工作最核心的一个对象，包含了构建的所有配置信息，也通过这个对象去注册钩子函数，明确了需求与任取之后，需要了解要用那个钩子，emit钩子在webpack即将王输出目录输出文件时执行。tap()方法注册钩子函数，接受两个参数，第一个为参数名称，二为挂载到钩子上的函数，compilation可以理解为此次打包的上下文，其assets属性获取即将写入目录当中的资源信息，是个对象.插件是通过在生命周期的钩子中挂载函数实现拓展。

5、webpack-dev-server打包、监听，

安装：***\*yarn add webpack-dev-server --dev\****

运行：***\*yarn webpack-dev-server或者在package.json文件夹加入script命令\****。

打包结果暂存内存，减少很多不必要的磁盘读写操作。

自动唤起浏览器指令：***\*yarn webpack-dev-server --open\****

6、webpack-dev-server静态资源访问。在webpack.config.js文件中加入：

***\*devServer：{\****

***\*contentBase:‘ ’或[],即字符串或数组，一个或多个路径，‘./public’\****

***\*}\****

contentBase作用为额外开发服务器指定查找资源目录。

Copywebpackplugin一般在开发阶段最好不要使用这个插件，等到项目上线前那次打包再用。

7、使用CORS的前提是api必须支持。但并不是任何情况api都应该支持，如果前后端同源部署，前后端部署端口、协议等都一致时就没必要支持。Devserver支持通过配置开启代理服务，网址：api.github.com

***\*devServer：{\****

***\*contentBase:‘ ’或[],即字符串或数组，一个或多个路径，‘./public’\****

***\*Proxy: {\****

***\*‘/api’: {\****

***\*Target: ‘https://api.github.com’\****

***\*‘pathRewrite’: {\****

***\*‘^/api’: ‘’\****

***\*},\****

***\*changeOrigin: true\****

***\*}\****

***\*}\****

***\*}\****

8、Source map

在原min.js最后一行中添加注释：//***\*# sourceMappingURL=XX.map\****指向原map文件

配置：在webpack.config.js文件中加入：***\*devtool: ‘source-map’\****

12种方式：

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps53E6.tmp.jpg) 

eval模式：是否使用eval执行模块代码。将模块代码放到eval（）函数中执行并且通过sourceURL去标注这个模块文件的路径，将***\*devtool设为：‘eval’\****，然后执行yarn webpack打包命令。***\*eval(’//# sourceURL=webpack:///./src/main.js’)\****中该注释指令只会定位文件，不会生成source map文件，只是定位那个文件出了错误。

eval-source-map模式：可以具体定位行和列的信息，相比eval，生成了source map文件。

cheap-eval-source-map模式：sourcemap是否包含行信息。阉割版，只能定位到行，没有列。Es6转换后的结果。

cheap-module-eval-source-map模式：是否能够得到loader处理之前的源代码。没有es6转换，接近源码。

hidden-souce-map模式：在开发工具中是看不到效果的。但源代码源文件夹中确实生成了一个hidden的文件。没有通过sourceURL注释引入文件。开发第三方包时很有用。

inline-souce-map模式：跟普通source-map文件一样，以dataurl形式嵌入代码中。最不可能用到的，会导致代码变得很大。

nosources-souce-map模式：能看到错误信息，但点进去看不到源代码，可以看到行信息，为了保护源代码。

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps53E7.tmp.jpg) 

***\*开发模式下选择cheap-module-eval-source-map模式，每行代码不超过80字符，可以快速定位到行就行了。首次打包速度慢无所谓，重写打包相对较快。\****

***\*生产模式选择none，nosources-souce-map不生成sourcemap。不暴露源代码。且调试是开发阶段的事情。不要生产模式全民公测。\****

不同模块对比：webpack配置对象可以是个数组，数组中的每个元素是一个单独的打包配置，这样可以在一次打包过程中，执行多个打包任务。如module.exports = [{entry:’’,output:{filename:’a.js’},{entry:’’,output:{filename:’b.js’}]就会有同时两个子任务工作，生成a.js和b.js文件

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps53F7.tmp.jpg) 

9、HMR，全程Hot Module Replacement模块热替换。解决整体刷新带来的页面更改变化丢失的问题。页面不刷新的前提下，模块也可以及时更新。热拔插，在一个正在运行的机器上随时插拔设备。模块热替换即在应用运行过程中实时替换某个模块，而应用运行状态不受影响。热替换只将修改的模块实时替换到应用中。HMR是webpack最强大的功能之一。

HMR集成在webpack-dev-server中，不需要单独安装模块了。***\*webpack-dev-server --hot指令就可开启，也可在配置文件两个地方设置devServer：{hot:true}；在开头导入webpack：const webpack = require（‘webpack’）后在plugins：[new  webpack.HotModuleReplacment\****

***\*Plugin()]。最后执行yarn webpack-dev-server --open\****

10、Webpack中的HMR并不能开箱即用。需要手动处理模块热替换逻辑。样式文件的热更新可以开箱即用，因为样式是通过loader更新的，style-loader已自动热更新。而js导入的模块是毫无规律的，无法实现通用的情况去处理对应更新。之所以框架中不用手动处理，是因为框架下的开发，每种文件都是有规律的。通过脚手架创建的项目都集成了HMR方案。总结：需要手动处理JS模块更新后的热替换。

11、HMR apis 打包入口文件中导入了其他的模块，从而模块更新了随时更新。***\*module.hot.accept(‘依赖模块路径’，处理函数)\****是注册模块更新后的处理函数。eg：module.hot.accept(‘./editor’,()=>{console.log(‘热更新了’)})。

12、处理JS模块热替换，没有统一固定的方式。针对文本类热更新，思路是热更新前先复制页面中更新的内容，然后等热更新时再复制该内容到页面上即可，具体代码见下：

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps53F8.tmp.jpg) 

13、图片模块热替换。只需将图片路径修改为新的路径即可，因为在图片修改后，图片文件名会发生变化，而新路径拿到的是更新过后的路径，eg，const img =new Image（）    

img.src = background    module.hot.accept(‘旧文件路径./better.png’,()=>{img.src = background  console.log(background)})

HMR注意事项：1、代码有问题时不易发现，也热更新，且刷新后错误信息被清除。解决办法：hot方式热替换失败，自动回退使用热刷新，而hotOnly不会自动刷新，在配置文件中修改hot：true为***\*devServer：{hotOnly：true}\****后再重新启动yarn webpack dev-server。再修改代码，无论代码是否使用了模块热替换，浏览器都不会自动刷新。2、没启用HMR的情况下，HMR apis会报错，没有accept函数，解决办法：法一，在配置文件加上***\*在plugins：[new  webpack.HotModuleReplacmentPlugin()]\**** 开启；法二，在accept函数之前加上if（module.hot）{正常hot.accept（）函数内容}；3、代码中多了很多跟生产没有关系的代码，如果配置文件没有配置HMR，运行yarn webpack打包后，打包生成的文件中会发现已自动移除处理模块热更新的代码，只剩下if（false）{}空判断，这种没有意义的判断，压缩后会自动删除。

14、生产环境注重运行效率。开发环境只注重开发效率。不同的工作环境（生产与开发环境）创建不同的配置的方法有两种：一种是配置文件根据环境不同导出不同配置，方法见下图：

先将开发环境的配置放到***\*module.exports = （env, args）=>{const config = {开发环境所有的配置内容放这里}   if(env===’production’){config.mode=’production’等等生产环境的配置项放这里}}\****  然后分别通过yarn webpack和yarn webpack --env production

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps5409.tmp.jpg) 

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps540A.tmp.jpg) 

一种是一个环境对应一个配置文件，适用于大型项目。至少有三个配置文件，两个是代表不同环境各自的配置（webpack.dev.js和webpack.prod.js），一个是两种环境共同的通用配置文件(webpack.common.js)。执行***\*yarn add webpack-merge --dev\****模块。此模块导出merge函数，用来合并公共配置和不同环境定义的配置。就不需要使用Object.assign和lodash中的match函数。打包时执行***\*yarn webpack --config webpack.prod.js(要打包的具体的文件名)，也可以把它放到package.json文件中“scripts”：{“build”：“webpack --config webpack.prod.js”} 然后执行yarn build就可以了。\****

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps540B.tmp.jpg) 

15、Webpack4中新增的production模式下内部自动开启很多优化功能。其中DefinePlugin为代码注入可能会发生变化的值或全局成员。Production模式下默认会启动且自动注入process.env.NODE_ENV。注意：值要求为js代码片段，如果直接字符串，放入JSON.stringify（）中就符合要求了。

16、Tree-shaking会摇掉未引用代码（dead-code）。Tree Shaking不是指某个配置选项，是一组功能搭配使用后的优化效果，会在production模式下自动开启。其他模式需要手动开启。方法为***\*添加optimization：{usedExports: true,  concatenateModules：true， minimize:true}\****。其中usedExports负责标记枯树叶，minimize负责摇掉他们。concatenateModules作用是尽可能的将所有模块合并输出到一个函数中。这样既提升了运行效率，又减少了代码的体积，这个特性又叫Scope Hoisting，作用域提升是webpack3提出的特性，再配合minimize代码的体积又会减小很多。

17、使用tree shaking的前提是使用ES Modules组织代码，即交给webpack打包的代码必须使用ES Module。webpack打包所有模块之前是将模块根据不同的配置交给不同的loader去处理，最后再将loader处理后的结果打包到一起，为了转换代码中的es新特性，一般选择babel-loader，如果使用了转换ESmodules的插件，（如配置了module：{rules：[{test:/\.js$/,use:{loader:’babel-loader’,options:{presets:[‘***\*@babel/preset-env\****’]}]},babel/preset-env）就会将ES Modules转换为CommonJS。配置module：{rules：[{test:/\.js$/,use:{loader:’babel-loader’,options:{***\*presets:[[‘@babel/preset-env’, { modules: ‘commonjs’ } ]]\****}]}会显性要babel-loader转为commonjs,注意是数组套数组，数组中再添加对象，对象内容为指明babel-loader将esmodule转换为commonjs，这样tree shaking就不会工作了。注意，最新版本的babel-loader默认关闭了esmodule转换为commonjs功能，可以在babeo-loader源码injectCaller.js和preset-env源码lib/index.js(下图)，这样webpack打包时最终得到的还是ESModule的代码，如果不确定到底有没有关闭，可以将上面的modules：false，就会关闭了。

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps540C.tmp.jpg)![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps541C.tmp.jpg) 

18、webpack4新增sideEffects副作用，允许通过配置的方式标识代码是否有副作用，从而为tree shaking提供更大的压缩空间。副作用是指模块执行时除了导出成员之外所做的事情，再开发一个npm模块时才会用到，用于npm包标记是否有副作用。***\*optimization：{sideEffects:true}\****，开启这个功能，production模式下自动开启，开启后打包时会先检查package.json中有没有sideEffects的标识，以此来判断该模块是否有副作用，没有副作用时，没有用到的模块不会打包，***\*在package.json中添加“sideEffects”：false\****，作用标识代码没有副作用的，再运行打包命令后，没有用到的模块就不会打包进来了。

19、使用sideEffects的前提是代码真的没有副作用，否则打包时会误删有副作用的代码。待重看补充。

20、Code Splitting代码分割。bundle代码打包到一起，体积过大，然后并不是每个模块在启动时都是必要的，把所有模块打包到一起，会加载所有模块才能使用，浏览器会浪费很多带宽和加载，解决方案是分包，按需加载。资源太大也不行，太碎也不行，HTTP1.1不能同域并行请求，每次请求都会有一定的延迟，请求的Header浪费带宽流量，模块打包是必要的，webpack支持分包，形成多个bundle.js,方法有两种：一多入口打包，适用于传统的多页应用程序，一个页面对应一个打包入口，公共部分单独提取，将配置文件中的***\*entry定义为对象{}，eg:entry:{index:‘./src/index.js’,album：‘./src/album.js’}\****.注意不是数组，数组是把多个文件打包到一起，还是一个入口，然后修改***\*output：{filename：‘[name].bundle.js’}\****,然后再到每个自动注入所有打包结果的htmlwebpackplugin插件实例中添加chunks指定所使用的bundle，每个打包入口都会形成独立的chunk：[‘打包生成的文件名，entry中的键名’]。Eg：plugins：{new HtmlWebpackPlugin({***\*chunks:[‘inde或者album’]\****})}，如想提取公共模块，添加***\*optimization：{splitChunks：{chunks：‘all’}}\****

，打包后会在dist目录中生成album-index.bundle.js；二采用ESModule的动态导入功能实现按需加载：需要用到某个模块时，再加载这个模块。所有动态导入的模块会被自动分包。方法为在index.js不要在开头全部import所有模块，而是在下面相应的模块用import()函数动态导入，注意该函数返回的是个promise，在该函数的then方法中可以拿到模块对象进行操作。整个过程无需任何配置，只需按ESMoudle的动态导入成员的方式即可，webpack自动处理分包和按需加载。如果使用的是单页应用框架，如vue和react，在项目中的路由映射组件就可以通过动态导入的方式实现按需加载。Eg:if(hash ===’#posts’){***\*import\****

***\*(/\* webpackChunkName:’album(打包后文件名开头部分)’\*/\****‘./post/posts’).then(([

default:posts])=>{mainElement.appendChild(posts())})}。其中webpackChunkName为魔法注释，用于设置打包后解决默认的由数字开头标识的文件名，而使用他后面的值为文件开头名。如果多个webpackChunkName设置一样的值，将打包到一起。

21、MiniCssExtractPlugin插件可以使css代码从打包文件中提取出来到单个文件，可以实现css模块的按需加载。用法：先安装yarn add mini-css-extract-plugin --dev，后到配置文件顶部导入该插件const MiniCssExtractPlugin = require（‘mini-css-extract-

plugin’）后在plugins：{new MiniCssExtractPlugin()}会自动提取CSS到一个单独的文件中，这相当于一个单独的css文件，所以不再需要style-loader了（因为style-loader是通过style标签注入文件的），而是用MiniCssExtractPlugin.loader替代，当然，如果css文件过小，为减少一次请求，没必要单独形成个文件。建议当CSS文件超过150Kb时，才考虑提取到单独一个文件。

22、webpack内置的压缩插件仅仅针对于js文件，其他资源文件需要单独安装插件，optimize-css-assets-webpack-plugin插件压缩样式文件。***\*先安装yarn add optimize-css-assets-webpack-plugin --dev，在配置文件导入插件const OptimizeCssAssetsWebpackPlugin = require(‘optimize-css-assets-webpack-plugin’)，后添加plugins：{new OptimizeCssAssetsWebpackPlugin() }\****。注意配置到plugins数组中，该插件任何时候都工作，而官网中配置到optimization：{minimizer:[new OptimizeCssAssetsWebpackPlugin()]}则只会在该命令开启时(即生产模式yarn webpack --mode production)才会工作，但是这样会存在一个问题，会导致js不压缩了，这是因为手动设置了minimizer时webpack会认为我们要自定义所使用的压缩插件，内部内置的js压缩器就会被覆盖掉，故需手动添加回来，方法：***\*先安装yarn add terser-webpack-plugin --dev模块，再到配置文件手动导入该插件 const TerserWebpackPlugin = require(‘ terser-webpack-plugin’),optimization：{minimizer:[new OptimizeCssAssetsWebpackPlugin(),new  TerserWebpackPlugin()]}\****，这样js文件也可以压缩了。

23、众所周知，浏览器资源文件给用户缓存会提升性能，但过期时间设置的过短效果不显著，过长则更新后没法及时更新到客户端，为了解决这个问题生产模式下，文件名使用hash哈希值，一旦资源文件改变，文件名称也跟着改变，对于客户端而言，全新的文件名意味着全新的请求，就没有缓存的问题，这样就可以把服务器缓存策略的过期时间设置的非常长，不用担心文件更新过后的问题。Webpack和绝大多数插件的filename属性都支持通过占位符的形式为文件设置哈希，有3种方法：1、***\*在filename中添加[hash]\****，最普通的整个项目级别的哈希，即项目中任何地方代码改动，项目中生成的文件名哈希值都会变化，eg：filename：‘[name]-[hash].bundle.js’。2、***\*在filename中添加[chunkhash]\****，属于chunk级别的哈希，即项目中只要是同一路（同一个文件夹，如post文件夹下的js和css区别其他路径下的js和css）的打包，chunk哈希都是相同的，相比普通哈希，更精确些。3、***\*在filename中添加[hash]\****，属于文件级别的哈希，最适合的最精准的哈希。即单个文件变动，只相应文件的哈希变动，根据文件内容生成哈希，不同的文件就有不同的哈希值。另外，可以在哈希名称后面加上冒号：和数字指定哈希长度，eg：filename：‘[name]-[contenthash:

8].bundle.js’，控制缓存8位的哈希是最好的选择。

24、eslint是最为主流的js lint工具，检测js代码质量，ESLint很容易统一开发者的编码风格，也可以帮助开发者提升编码能力。安装步骤：

***\*npm init –yes\****用于管理项目的依赖，生成package.json，

***\*npm install eslint –save-dev\****（eslint提供cli）

cd.\node_modules\.bin\然后执行.***\*\eslint –version 或(cd../../)用npx/yarn eslint --version\****检查安装成功与否及版本

25、编写好代码后，Eslint使用方法:

1、***\*npx eslint -–init\****，第一个问题选择最后一个最长的检查代码语法、问题代码及风格第二个问题语法风格，ESModule、commonjs和none，第三个选择有无框架，第四个选择ts，第五个代码运行在什么环境（浏览器或node），第六个选择怎样的代码风格，选市场上主流风格，standard不需要分号，第七个问题配置文件以何种格式存放，选js，最后一个问题，安装额外插件，选yes，npm会自动安装。一切就绪后，项目根目录下会多出.eslintrc.js的eslint配置文件。

2、执行npx eslint+路径或路径通配符,eg:npx eslint .01-prepare.js,eslint会第一步检查语法错误代码，不会检查其他问题，修正好问题代码后，继续执行上面命令接着检查问题代码，检查出来后执行npx eslint .01-prepare.js –fix命令就会自动修正问题代码，有些问题还是不能检查出来，需要手动检查删除，然后再执行npx eslint .01-prepare.js –fix命令，直到没有任何报错了。

26、eslint配置注释。出现一两个需要违反设置的规定时，为了不影响全局设定的规定，可以用配置注释，很多规则具体参考eslint官方文档http://eslint.cn/docs/user-guide/

configuring#configuring-rules。如在某一行代码后添加//eslint-disable-line，但一行有多个错误时都会忽略，不会检查到了，故在后面跟上具体要禁用的规则名称，//eslint-disable-line no-template-curly-in-string。

27、eslint结合自动化工具gulp，先打开gulpfile.js，然后按***\*ctrl+k再ctrl+0\****(数字0)折叠所有代码，找到处理js文件的gulp指令，因为gulp是管道的处理方式，在babel处理前加入eslint，否则经过babel处理后代码就不是源代码了，安装且用init指令生成.eslintrc.js的eslint配置文件，然后在gulp.js导入eslint模块，如果用plugin代替，就不需要引入了，在使用npx/yarn gulp XXX执行，默认情况下，eslint只检查代码中的问题，不会对错误处理结果有任何反馈，使用eslint的format()方法，在控制台打印错误信息，failAfterError()检查出错误代码后自动终止管道，这样eslint集成到编译js任务当中了。如果出现$不匹配的问题，在.eslintrc.js中加入globals：{‘$’:‘readonly’}

![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps541D.tmp.jpg) 

28、webpack是通过loader集成eslint的，非插件，一定要注意eslint-loader是放到js的loader的最后，率先执行。也可以为js文件重新添加***\*loader，{test：/\.js$/,exclude:/node_module/,use:’eslint-loader’,enforce:’pre’\****},然后执行npx webpack即可自动检查代码中的问题。针对react，安装以下插件：npm install eslint-plugin-react，然后在..eslintrc.js中添加rules：[‘react/jsx-uses-react’:2，‘react/jsx-uses-vars’：2],plugins：[‘react’],plugins是个数组，专门放置安装的插件,这里里面之所以写成’react’，是因为会自动去除前面的eslint-plugin，要开启某个规则，可以将它的值设为aver（根据老师发音标记的），可以用2替代，然后执行npx webpack。React的app没有定义配置react/jsx-uses-react。对于大多数eslint插件来说，一般提供共享的配置，降低使用成本，上面两个插件提供了共同的配置recommended和all，即可以在extends:[‘plugin:react/recommended’]继承，写法为plugin后加冒号，在写要继承的插件名后斜线添加具体的配置名字。注释之前的plugins和rules中的代码即可。

29、Vue和react不需要操作eslint和webpack，自动集成了。Npm install @vue-cli -g

30、Eslint检查ts。记得先安装ts，npx eslint --init选择ts时为yes，然后在.eslintrc.js中添加***\*parser：’@typescipt-eslint/parser’\****,parser的作用是语法解析器，ts有很多特殊的语法，需要指定语法解析器，完成这些步骤后就会自动可以使用eslint检查ts了。

31、Stylelint用于校验css代码，提供默认的代码检查规则，也提供cli工具，终端快速调用，通过插件支持sass less postcss，也支持gulp或webpack集成。初始化eslint后，安装stylelint：***\*npm install stylelint -D,安装npm install stylelint-config-sandard\****用于将来单独安装继承的stylint共享模块,然后手动添加.stylelintrc.js，内容：***\*module.exports = {extends：”stylelint-config-standard”}\****,extends相比.eslintrc.js不同的是需要写全称，***\*npx stylelint ./index.css\****即可。要校验sass文件，安装***\*npm install stylelint-config-sass-guidelines -D\****,然后在.stylelintrc.js修改***\*extends:[‘stylelint-config-standard’,’stylelint-config-sass-guidelines’]\****既可以了，执行npx stylelint ./index.sass。

32、Prettier前端代码自动格式化，包括.js、.json、.html、.css、.scss、.vue、.jsx、.md，先安装：***\*npm install prettier,然后执行npx prettier style.css\****，默认情况下将格式化后代码打印在控制台中，要想直接覆盖源文件，执行***\*npx prettier style.css --write\****。***\*npx prettier . write\****会格式化根目录下的所有文件且覆盖到源文件上。不要完全依赖工具，日常写代码时就注意使用规范。

33、Git hooks与eslint的结合。团队中有人代码提交至仓库前未执行lint工作时，需要通过git hooks在代码提交前强制lint。Git hooks也成为git钩子，每个钩子都对应一个任务，通过shell脚本可以编写钩子任务触发时要具体执行的操作。定义一个git仓库目录git init，打开.git文件夹里面的hooks文件夹里的每个.sample就是一个钩子，源目录复制一份pre-commit.sample文件，删除复制后的文件后缀名.sample，这样就可以去编写这个文件了，打开后留下决定代码运行环境的首行#!/Bin/sh，其余全删掉，编写shell脚本。如echo“输出的内容”，通过git编辑器执行touch demo.txt后vim demo.txt，后git status后git add.后git commit -m“内容”就会提示之前文件里面的echo命令。

34、对不擅长使用shell的开发者，可安装husky模块（用之前记得删除之前复制新建的钩子文件）：***\*npm install husky -D\****,可找到仓库中的.git/hooks目录会多出一些自定义的钩子，找到项目中根目录的package.json添加***\*“scripts”：{“test”：“echo \”Error: not test specified””}“husky”：{“hooks”：{“pre-commit”：“npm （run） test”}}\****，将来执行commit提交时，就直接找到scripts下的test里设置的代码操作，在控制台先执行git add.后执行git commit -m “内容”。要真正实现commit前做eslint检查，***\*修改”scripts”：{“test”：“eslint ./index.js”}\****,执行git add .后执行git commit -m “”后就会显示eslint的检查内容，若想检查后进一步自动操作，需安装***\*npm install lint-staged -D\****，在package.json中加入***\*“lint-staged”：{“\*.js”:[“eslint”,”git add”]},”scripts”:{“precommit”:”lint-staged”},把“husky”\****里的***\*“pre-commit”：“npm run precommit”\****的修改，最后执行git add .和git commit -m “”后终端可以看到先后执行了eslint和git add的操作，实现在commit之前强制执行eslint。推荐重点搭配使用husky和lint-staged。



![img](file:///C:\Users\qinhe\AppData\Local\Temp\ksohtml\wps541E.tmp.jpg)



另附相关知识：

、yarn Workspace 使用指南

Yarn 从 1.0 版开始支持 Workspace （工作区）。

Workspace 能更好的统一管理有多个项目的仓库，既可在每个项目下使用独立的 package.json 管理依赖，又可便利的享受一条 yarn 命令安装或者升级所有依赖等。更重要的是可以使多个项目共享同一个 node_modules 目录，提升开发效率和降低磁盘空间占用。一句话总结就是可以大大简化对多个项目的统一管理。

很多知名的开源项目也使用了 [Yarn Workspace](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspace:)，如 [vue](https://links.jianshu.com/go?to=https://github.com/vuejs/vue-next)、[react](https://links.jianshu.com/go?to=https://github.com/facebook/react)、[jest](https://links.jianshu.com/go?to=https://github.com/facebook/jest) 等。

\1. [Yarn Workspace](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspace:) 共享 node_modules 依赖详解

回看下面的带两个子项目的经典 Node.js 项目：

projects/

|--project1/

|  |--package.json

|  |--node_modules/

|  |  |--a/

|--project2

|  |--package.json

|  |--node_modules/

|  |  |--a/

|  |  |--project1/

project1/package.json:

{

 "name": "project1",

 "version": "1.0.0",

 "dependencies": {

  "a": "1.0.0"

 }}

project2/package.json:

{

 "name": "project2",

 "version": "1.0.0",

 "dependencies": {

  "a": "1.0.0",

  "project1": "1.0.0"

 }}

没有使用 [Yarn Workspace](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspace:) 前，需要分别在 project1 和 project2 目录下分别执行 yarn|npm install 来安装依赖包到各自的 node_modules 目录下。或者使用 yarn|npm upgrade 来升级依赖的包。

这会产生很多不良的问题：

如果 project1 和 project2 有相同的依赖项目 a，a 都会各自下载一次，这不仅耗时降低开发效率，还额外占用重复的磁盘空间；当 project 项目比较多的时候，此类问题就会显得十分严重。

如果 project2 依赖 project1，而 project1 并没有发布到 npm 仓库，只是一个本地项目，有两种方式配置依赖：

使用相对路径（如 file: 协议）在 project2 中指定 project1 的依赖。

使用 yarn|npm link 来配置依赖。

第 1 种方式缺少版本号的具体指定，每次发布版本时都需要相应的依赖版本的修改；第 2 种方式需要自行手工操作，配置复杂易出错。

需要 npm-2.0.0+ 才支持模块间的相对路径依赖，详见 npm 官方文档 [package.json/Local Paths](https://links.jianshu.com/go?to=https://docs.npmjs.com/files/package.json%23local-paths)

没有一个统一的地方对全部项目进行统一构建等，需要到各个项目内执行 yarn|npm build 来构架项目。

使用 [Yarn Workspace](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspace:) 之后，上述问题都能得到很好的解决。而且这是 Yarn 内置的功能，并不需要安装什么其他的包，只需要简单的在 projects 目录（Yarn 称之为 workspace-root）下增加如下内容的 package.json 文件即可。

projects/package.json：

{

 "private": true,

 "workspaces": ["project1", "project2"] // 也可以使用通配符设置为 ["project*"]}

开源社区则都基本上使用 "workspaces": ["packages/*"] 的目录结构，这与 [Lerna](https://links.jianshu.com/go?to=https://github.com/lerna/lerna) 的目录结构一致。

在 workspace-root 目录下执行 yarn install：

$ cd projects

$ rm -r project1/node_modules

$ rm -r project2/node_modules

$ yarn install

yarn install v1.22.0

info No lockfile found.

[1/4] 

此时查看目录结构如下：

projects/

|--package.json

|--project1/

|  |--package.json

|--project2

|  |--package.json

|--node_modules/

|  |--a/

|  |--project1/ -> ./project1/

说明：

projects 是各个子项目的上级目录，术语上称之为 workspace-root，而 project1 和 project2 术语上称之为 workspace。

yarn install 命令既可以在 workspace-root 目录下执行，也可以在任何一个 workspace 目录下执行，效果是一样的。

如果需要某个特殊的 workspace 不受 [Yarn Workspace](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspace:) 管理，只需在此 workspace 目录下添加 .yarnrc 文件，并添加如下内容禁用即可：

workspaces-experimental false

在 project1 和 project2 目录下并没有 node_modules 目录（特殊情况下才会有，如当 project1 和 project2 依赖了不同版本的 a 时）。

/node_modules/project1 是 /project1 的软链接，软链接的名称使用的是 /project1/package.json#name 属性的值。

如果只是修改单个 workspace，可以使用 --focus 参数来快速安装相邻的依赖配置从而避免全部安装一次。

\2. 可用的 [Yarn Workspace](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspace:) 命令

2.1. [yarn workspace  ](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspace)

针对特定的 workspace 执行指定的 <command>，如：

$ yarn workspace project1 add vue --dev 《 往 project1 添加 vue 开发依赖

$ yarn workspace project1 remove vue   《 从 project1 移除 vue 依赖

在 {workspace}/package.json#scripts 中定义的脚本命令，也可以作为 <command> 来执行。

下面是一个利用这个特点创建统一构建命令的例子：

projects/package.json:

{

 "scripts": {

  "build": "yarn workspaces run build"

 }}

project1|project2/package.json:

{

 "scripts": {

  "build": "rollup -i index.js -f esm -o dist/bundle.js"

 }}

执行 yarn build 的结果：

$ yarn build

yarn run v1.22.0

$ yarn workspaces run build

\> project1

$ rollup -i index.js -f esm -o dist/bundle.js

index.js → dist/bundle.js...

created dist/bundle.js in 70ms

\> project2

$ rollup -i index.js -f esm -e project1 -o dist/bundle.js

index.js → dist/bundle.js...

created dist/bundle.js in 80ms

✨  Done in 2.45s.

2.2. [yarn workspaces ](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspaces)

2.2.1. yarn workspaces run <command>

在每个 workspace 下执行 <command>。如：

yarn workspaces run test

将会执行各个 workspace 的 test script。

2.2.2. yarn workspaces info [--json]

显示当前各 workspace 之间的依赖关系树。

$ yarn workspaces info

yarn workspaces v1.21.1{

 "project1": {

  "location": "project1",

  "workspaceDependencies": [],

  "mismatchedWorkspaceDependencies": []

 },

 "project2": {

  "location": "project2",

  "workspaceDependencies": [

   "project1"

  ],

  "mismatchedWorkspaceDependencies": []

 }}

104、✨  Done in 0.12s.

105、相关源代码已放在 Github 上，详见[这里](https://links.jianshu.com/go?to=https://github.com/start-nodejs/yarn-workspace)。

Yarn Workspace 官方参考文档：[英文](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/en/docs/cli/workspace)、[中文](https://links.jianshu.com/go?to=https://classic.yarnpkg.com/zh-Hans/docs/workspaces)。

 