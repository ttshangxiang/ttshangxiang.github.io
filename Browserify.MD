
1，简介

Browserify 可以让你使用类似于 node 的 require() 的方式来组织浏览器端的 Javascript 代码，通过预编译让前端 Javascript 可以直接使用 Node NPM 安装的一些库。

2，官方举例

npm install -g browserify

var unique = require('uniq');

var data = [1, 2, 2, 3, 4, 5, 5, 5, 6];

console.log(unique(data));

npm install uniq

browserify main.js -o bundle.js

<script src="bundle.js"></script>

3，原理
browserify实现了CommonJS的api，能将require依赖的代码连同CommonJS运行环境打包到文件。
打包前文件
name.js
  module.exports = "aya";
main.js
  var name = require("./name");
  console.log("Hello! " + name);
打包后的文件基本结构如下
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

这是一个自执行函数，函数体部分为CommonJS的运行时，主体部分是参数。
第一个参数是代码模块，每个模块被分配一个数字，对象的属性为一个数组，第一个是模块主体，第二个是它的依赖。
第二个参数表示本次编译之前被加载进来的来自于其他地方的内容。
第三个参数表示指定的是作为入口的模块id，可以有多个。

详细的实现过程分析请参考https://segmentfault.com/a/1190000004128257

4，应用
有两种应用方法

1，敲指令

browserify main.js -o bundle.js

2，使用api

结合gulp使用一般是这样的
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
    .pipe(gulp.dest('./dist/js'));
});

或者
在 gulp 管道中使用 throuth2 操作 vinyl 文件对象，browserify 处理以后再返回管道中。

var browserify = require('browserify');
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var through2 = require('through2');

gulp.task('browserify', function() {
  return gulp.src('./src/js/app.js')
    .pipe(through2.obj(function(file, enc, next) {
      browserify(file.path)
        // .transform(reactify)
        .bundle(function(err, res) {
          err && console.log(err.stack);
          file.contents = res;
          next(null, file);
        });
    }))
    .pipe(uglify()) // uglify
    .pipe(gulp.dest('./dist/js'));
});

b.require(file, {expose: 'alias'})可指定依赖别名
b.external(file) 可忽略

throuth2在gulp中很好用，使用throuth2可以拿到文件的名称和内容等，所有有需要也可以修改文件的内容。

通过plugin进行指定外部依赖包等操作。
通过transform可进行ES6/ts/jsx等的转换。

当依赖的文件很多时，browserify 处理速度会很慢，使用 watchify 可以大幅提高处理速度。原理是browserify每次都会完整编译，而通过watchify只重新编译有变化的部分。


5，与webpack的比较。
webpack是现在最常见的构建工具，通过配置，它能打包js/css/html/图片等几乎所有的资源，非常方便。但当出未知的错误时，如果对配置项不是很熟悉，排查起来会很麻烦。

相比起来，gulp+browserify的组合不是需要你配置，而是写代码。它没有webpack那么自动化，所以每个任务都需要你亲自安排，相对灵活，可定制化程度高，比较适合对自己的代码有自信的人。通过各种插件，这个组合也能很好的实现前端的构建。

综合考虑，一般项目我还是推荐webpack，哈哈哈。

6，参考
browserify官网
http://browserify.org/
百度百科
https://baike.baidu.com/item/browserify/16823990?fr=aladdin
gulp中文网
https://www.gulpjs.com.cn/
解析browserify工作原理：
https://segmentfault.com/a/1190000004128257
使用Browserify来实现CommonJS的浏览器加载：
https://www.cnblogs.com/xiaohuochai/p/6850977.html
基于 Gulp + Browserify 构建 ES6 环境下的自动化前端项目：
https://segmentfault.com/a/1190000004917668
在 Gulp 中使用 Browserify
https://csspod.com/using-browserify-with-gulp/
使用Browserify配合jQuery进行编程的超级指南
https://www.jb51.net/article/70164.htm