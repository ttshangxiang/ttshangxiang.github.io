## 我用到过的ES6
es6内容很多，但是我用到的基本上就这些。
1. let，const。  
基本上就是，引用库用const，定义变量用let。const不能再被赋值，但如果他是对象，那他的内部属性能被改变。
2. 解构，扩展。
```
let a = 1;
let b = 2;
// 对象省略写法，key和变量名一致可以这样写
let obj = {a, b}; 
// 这是解构，很多时候用于import
let {a, b} = obj;
// 这这是扩展
let obj2 = {
    c: 3,
    ...obj
}
```
3. 函数写法简化。
```
// 箭头函数，箭头无this，this是外层的，很方便
let a = (a) => {return a;}
// 简写
let a = a => a;
// 在类里面
class Class {
    a () {
        return 0;
    }
}
```
4. 默认值。
```
//解构默认
let {a = 0, b} = obj;
//函数默认
function fun(a = 0, b) {...}
```
5. export，import。
```
// 普通导出
export default let a = 0;
// 默认导出
export let a = 0;
// 默认导入
import a from './a';
// 普通导入
import {b} from './a';
import * as a from 'a';
// 混用
import a, {b} from './a';
```

## lodash中好用的方法
我发现webpack都在用lodash，让我觉得这个库很可靠。大部分内容es6都有，但是现在还是可以用一下。
* _.assign()  
可用于复制、合并对象，不是深拷贝，只拷贝一层。
```
_.assign({}, a, b); // 经常这样用
```
* _.get()  
按照路径获取数据，可以设置返回值，数据结构深且不可信的时候超有用，不用担心报错，不需要一层一层if判断是否存在。
```
_.get(obj, 'a.b.c.d', null);
```
* _.find()  
从数组或对象中按条件查找对象    
```
_.find(users, function(o) { return o.age < 40; });
```
* _.times()  
执行多次
```
_.times(3, Number)// [0, 1, 2]
_.times(3, Number)// ['0', '1', '2']
_.times(3, function() {return 1;});// [1, 1, 1]
```
* _now()  
现在的时间戳
```
_now(); //1497509692736
```
* _.maxBy()   
找最大
```
_.maxBy(list, function(o){return o.a;}); //返回数组中a最大的
```
* _.keyBy()  
数组转为对象，设置一个属性为key
```
_.keyBy(list, 'a'); //返回对象以a的值为key
```
* _.filter()  
过滤数组
```
_.filter(list, function(o){return o.a == 1;}); //过滤掉a为1的
```
* _.includes()  
是否包含，对象数组字符串都可以用
```
_.includes([1, 2, 3], 1);// => true
_.includes([1, 2, 3], 1, 2);// => false
_.includes({ 'a': 1, 'b': 2 }, 1);// => true
_.includes('abcd', 'bc');// => true
```
* _.chain()  
这个我没用过，但是看上去很有用，打一个包，然后可以链式调用，这个包有个value方法，用于计算，当需要对数组做一系列操作时可以用    
```
var users = [
  { 'user': 'barney',  'age': 36 },
  { 'user': 'fred',    'age': 40 },
  { 'user': 'pebbles', 'age': 1 }
];
 
var youngest = _
  .chain(users)
  .sortBy('age')
  .map(function(o) {
    return o.user + ' is ' + o.age;
  })
  .head()
  .value();
// => 'pebbles is 1'
```

## 以前遇到过的问题
* IE下数据接口不刷新数据  
在IE下，get接口如果链接没发生改变，不会发送请求，而是拿缓存，即使是edge也是一样。需要在链接后加时间戳或者随机数来触发请求，jquery可以这样写：  
```
$.ajaxSetup ({
    cache: false //关闭AJAX相应的缓存
});
```
* IE8下document.referrer为null  
以前做登录返回的时候遇到的，通过window.location.href = '/path'这样的跳转，IE8下来源是不存在的，必须构造一个a标签设置href，模拟点击才行。
* IE8下for in的问题  
IE8下，for in时，如果不用hasOwnProperty过滤，Array会遍历length，Object会遍历toString，相当可怕。
* Vue中改变数据后，立刻获取dom的高度，发现是上次的高度  
Vue中数据修改后，不会立刻渲染dom，而是要等所有数据改变完才渲染，所以获取dom数据时根本还没渲染，应该用this.$nextTick()在回调函数里面写，或者setTimeout(fn, 0)，我发现在angular中也适用。
* 用fetch时，获取不到cookie  
在用fetch或fetch的兼容包时，参数里面一定要加credentials:"include"，才能发送cookie。
* 在js里面插入字符串dom时闭合标签没写/ 
这个问题听起来很简单，实际上很坑人。因为在html里面写标签时，一般都是补全，写错了也有提醒，在js里面忘了写真的很难定位bug。
* 

## 关于下拉框
1. 点击对话框外面关闭对话框，比较好看的写法，网上看的  
```
var func = function (event) {
    var e = event || $window.event;   
    var elem = e.srcElement||e.target;   
    while(elem)   
    {   
        if(elem.id = 'myDialog'){   
            return;   
        }   
        elem = elem.parentNode;        
    }
    document.removeEventListener('mousedown', func);
};
document.addEventListener('mousedown', func);
```
2. Element.matches()，dom是否被这个css表达式包含 
MDN上找到的，兼容不太好，但是有polyfill
如果元素将被指定的选择器字符串选择，Element.matches()方法返回true; 否则返回false。  
var result = element.matches(selectorString);
第一条的写法可以用这个写
```
var func = function (event) {
    var e = event || $window.event;   
    var elem = e.srcElement||e.target;   
    if (elem.id = 'myDialog' || elem.matches('#myDialog *')) {
        return;
    }
    document.removeEventListener('mousedown', func);
};
document.addEventListener('mousedown', func);
```
* 弹出框位置
我需要一个不会被overflow遮挡,会随滚动条滚动的下拉弹出框。
不被遮挡选择插入到body里面，用absolute布局，这样就出现一个问题，局部滚动条滚动时，他不跟着动。当时我的页面有两个滚动条，所以我监听这两个滚动条，在滚动时把下拉框关闭，这样做很不合适。
后来看element-ui的源码时，想看看他们的select是怎么写的，解决了我的问题，发现了他们用了一个叫popper.js的插件，我试用了一下，可以。
```
scope.popper = new window.Popper(element , getDom()[0], {
    placement: 'bottom'
}); //设置
scope.popper.update(); //更新
```
可以。
