## 一、简介
Browserify是一个Javascript预处理器，可以让你使用类似于node的require()的方式来组织浏览器端的Javascript代码，还可以通过预编译让前端Javascript直接使用Node NPM安装的一些库。  

随着浏览器性能的提高，前端可以做的事情也越来越多，这也意味着前端的代码量会上升，一个页面需要依赖大量的js，管理起来相当复杂，这个时候我们就需要用到模块化管理工具。  
现在构建react、vue项目时，大多会选择webpack，browserify可以是另一个选择。

## 二、官方示例
安装browserify：
```
npm install -g browserify
```
main.js  
在main.js中引入npm库uniq，进行数组去重：
```
var unique = require('uniq');
var data = [1, 2, 2, 3, 4, 5, 5, 5, 6];
console.log(unique(data));
```
安装uniq包：
```
npm install uniq
```
打包main.js，输出成bundle.js：
```
browserify main.js -o bundle.js
```
将bundle.js应用到html：
```
<script src="bundle.js"></script>
```

## 三、使用API
除了使用命令打包代码，还可以使用api：
```
browserify('./src/js/app.js')
.bundle()
.pipe(fs.createWriteStream('./bundle.js'));
```

### 常用api方法
var b = browserify('./src/js/app.js');
- b.add(file, opts)，添加入口文件。
- b.require(file, opts)，添加依赖，可添加第二个参数{expose: 'angular'}，设置别名。
- b.bundle(cb)，执行打包。
- b.external(file)，防止文件被加载到当前包中，而是从另一个包中引用。
- b.ignore(file)，防止模块名称或文件显示在输出包中，会用一个module.exports = {}代替。
- b.exclude(file)，防止模块名称或文件显示在输出包中。排除后，引用会报错。
- b.transform(tr, opts={})，在调用解析源代码之前对其进行转换，如Babel/Typescript/Jsx等转换成es5。
- b.plugin(plugin, opts)，加载插件。
- b.pipeline，可以在默认管道流中插入或删除自定义管道流。
- b.reset(opts)，将管道重置为正常状态。bundle()多次调用时会自动调用此函数。

更多请参见官网。

## 四、功能分析

### 内部过程：  
Browserify使用了Pipeline的思想，将其拆解为职责单一的多个子过程，例如分析依赖、拓扑排序、模块去重、打包合并等，并通过 Stream Pipeline 打通整个流程。这使得整个代码的架构异常清晰，对将来的维护和优化提供了良好基础。  
从以下这段核心代码可以看出处理的流程。  
```
var splicer = require('labeled-stream-splicer')
var pipeline = splicer.obj([
  // 记录输入管道的数据，重建管道时直接将记录的数据写入。
  // 用于像watch时需要多次打包的情况
  'record', [ this._recorder() ],
  // 依赖解析，预处理
  'deps', [ this._mdeps ],
  // 处理JSON文件
  'json', [ this._json() ],
  // 删除文件前面的BOM
  'unbom', [ this._unbom() ],
  // 删除文件前面的`#!`行
  'unshebang', [ this._unshebang() ],
  // 语法检查
  'syntax', [ this._syntax() ],
  // 排序，以确保打包结果的稳定性
  'sort', [ depsSort(dopts) ],
  // 对拥有同样内容的模块去重
  'dedupe', [ this._dedupe() ],
  // 将id从文件路径转换成数字，避免暴露系统路径信息
  'label', [ this._label(opts) ],
  // 为每个模块触发一次dep事件
  'emit-deps', [ this._emitDeps() ],
  'debug', [ this._debug(opts) ],
  // 将模块打包
  'pack', [ this._bpack ],
  // 更多自定义的处理
  'wrap', [],
])
```

### 准备文件：
打包前文件
- name.js
```
  module.exports = "aya";
```
- main.js
```
  var name = require("./name");
  console.log("Hello! " + name);
```
### module-deps
通过module-deps解析模块的依赖，输出成json：
```
[
  {
    "id": "1",
    "source": "var name = require(\"./name\");console.log(\"Hello! \" + name);",
    "deps": { "./name": "2" },
    "entry": true
  },
  {
    "id": "2",
    "source": "module.exports = "aya";",
    "deps": {}
  }
]
```
### browser-pack
通过browser-pack模块将上述的json数据打包合并成一个文件：
```
(function e(t, n, r) {
 // ...
})({
 1: [function (require, module, exports) {
  var name = require("./name");
 
  console.log("Hello! " + name);
 }, {"./name": 2}],
 2: [function (require, module, exports) {
  module.exports = "aya";
 }, {}]
}, {}, [1])
```
这是一个自执行函数，函数体部分为CommonJS的运行时，主体部分是参数。
- 第一个参数是代码模块，每个模块被分配一个数字，对象的属性为一个数组，第一个是模块主体，第二个是它的依赖。
- 第二个参数几乎总是空的{}。它如果有的话，也是一个模块map，表示本次编译之前被加载进来的来自于其他地方的内容。
- 第三个参数表示指定的是作为入口的模块id，可以有多个。

更详细的介绍可以参见：
https://github.com/browserify/browserify-handbook

## 五、结合gulp使用

结合gulp使用一般是这样的
```
var browserify = require('browserify');
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var source = require('vinyl-source-stream');
var buffer = require('vinyl-buffer');

gulp.task('browserify', function() {
  return browserify('./src/js/app.js') // 入口
    .bundle()
    .pipe(source('bundle.js')) // 出口
    .pipe(buffer())
    .pipe(uglify()) // 压缩
    .pipe(gulp.dest('dist/js'));
});
```

### 同时打包多个文件
当要同时打包多个文件，我们可以约定好入口文件格式，比如xx.entry.js，然后通过以下代码实现：
```
var browserify = require('browserify');
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var through2 = require('through2');

return gulp.src('./src/js/**/*.entry.js')
  .pipe(through2.obj(function(file, enc, next) {
    browserify(file.path)
    .bundle(function(err, res) {
      err && console.log(err.stack);
      file.contents = res;
      next(null, file);
    });
  }))
  .pipe(gulp.dest('dist/js/'))
```

### 速度优化  
当依赖的文件很多时，browserify 处理速度会很慢，使用 watchify 可以大幅提高处理速度。原理是browserify每次都会完整编译，而通过watchify只重新编译有变化的部分。
```
// 在这里添加自定义 browserify 选项
var customOpts = {
  entries: ['./src/index.js'],
  debug: true
};
var opts = assign({}, watchify.args, customOpts);
var b = watchify(browserify(opts));

// 下面就和browserify一样了
b.bundle()
......
```


## 六、与webpack的比较
webpack是现在最常见的构建工具，通过配置，它能打包js/css/html/图片等几乎所有的资源，非常方便。但当出未知的错误时，如果对配置项不是很熟悉，排查起来会很麻烦。

相比起来，gulp+browserify的组合不是需要你配置，而是写代码。它没有webpack那么自动化，所以每个任务都需要你亲自安排，足够灵活，可定制化程度高，比较适合对自己的代码有自信的人。通过各种插件，这个组合也能很好的实现前端的构建。

综合考虑，一般项目我还是推荐webpack。当webpack不能满足你的需求，或者你只需要一个简单的js模块化工具，使用browserify吧。

## 七、参考
- browserify  
https://github.com/browserify/browserify
- gulp中文网  
https://www.gulpjs.com.cn/
- 解析browserify工作原理  
https://segmentfault.com/a/1190000004128257
- 前端模块及依赖管理的新选择：Browserify  
https://segmentfault.com/a/1190000002941361
- 前端中的 Pipeline  
https://zhuanlan.zhihu.com/p/28561932
- Node.js Stream - 实战篇  
https://zhuanlan.zhihu.com/p/21681134
- browserify-handbook  
https://github.com/browserify/browserify-handbook

## 八、结尾
文章到此结束，由于水平和经验有限，如有纰漏或建议，欢迎留言。如果觉得不错，欢迎关注海致星图，谢谢您的阅读。