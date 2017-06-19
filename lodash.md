# 分享lodash中好用的方法

## 背景
我打算了解lodash是在看webpack官方例子的时候，发现他们在用，觉得或许是个好东西。  
  
lodash一开始是Underscore.js库的一个fork，因为和其他(Underscore.js的)贡献者意见相左。John-David Dalton的最初目标，是提供更多“一致的跨浏览器行为……，并改善性能”。之后，该项目在现有成功的基础之上取得了更大的成果。最近lodash也发布了3.5版，成为了npm包仓库中依赖最多的库。它正在摆脱屌丝身份，成为开发者的常规的选择之一。 
  
以下我就介绍几个我使用过、觉得有用的方法，希望给大家一些帮助。

## 链式调用、延迟计算：_.chain  
包装对象，通过链式调用，进行复杂的多步操作。  
chain是延迟执行的，只有调用value方法，才会真正计算，所以性能很好。
例子：假设我的需求渲染一个表格，取sort排名前三个，然后data1大于1显示红色，小于1显示绿色
```
var list = [
    {'company': 'a', 'data': 0.5, 'sort': 3},
    {'company': 'b', 'data': 1.2, 'sort': 1},
    {'company': 'c', 'data': 1.5, 'sort': 2},
    {'company': 'd', 'data': 0.9, 'sort': 4}
];
var list2 = _
    .chain(list)
    .sortBy('sort')
    .take(3)
    .map(function(o) {
        if (o.data > 1) {
            o.data = '<span style="color:red;">' + o.data + '</span>';
        }
        if (o.data < 1) {
            o.data = '<span style="color:green;">' + o.data + '</span>';
        }
        return o.data;
    })
    .value();
```
## 安全获取数据：_.get  
前端工程师肯定都遇到过这个的情况，后端给的数据不健全，json中有些数据可能会不存在，这时如果不经过判断，正常获取数据，可能会引发'Cannot read property 'xxx' of undefined'。  
这种时候一般的做法是一层一层的用if判断，写出类似“if (data && data.data && data.data.list && data.data.list[0])”这种代码，不是很优雅，这个时候_.get就可以派上用场了。
例子：
```
var item = _.get(data, 'data.list[0]', {});
// 第三个参数为没取到数据时的默认值。
```
## 扩展对象：_.assign  
我们在写类的构造函数时，经常会有这种场景：我有一些默认配置属性，在使用的时候还允许传入新的配置属性，来覆盖默认配置属性。如果我们没有其他库，可能需要写个遍历：
```
for (var i in config) {
    if (default.hasOwnproperty(i)) {
        default[i] = config[i];
    }
}
// 这个时候，我们也可以用assign
default = _.assign(default, config);
```
### 一些扩展
1. 关于类似的方法  
    jquery中：$.extend(default, config);  
    ES6中：Object.assign(default, config);

2. 以上方法与_.assign的异同  
    在lodash3版本，以上方法与_.assign特性相同。  
    在ladash4版本，以上两种方法基本上等同于_.assignIn，新版的_.assign不会改变第一个对象的值。

3. 关于使用时，为什么经常第一个参数写一个‘{}’
    在使用lodash3版本_.assign、$.extend()和Object.assign时，源数据也会被修改，这个会造成污染，没办法获取到源数据，所以一般都这样用：
    ```
    var obj = _.assign({}, default, config);
    ```
4. 关于引用  
    assign方法只会拷贝一层，给更深层次的属性赋值会有引用，如果对引用不是很了解，会导致产生一些bug，例子：
    ```
    var obj = _.assign({}, default, config);
    obj.data.name = 'a'; // 这样config或default里的数据也会被修改
    obj.data = {name: 'a'}; //这样才不会修改其他的值
    obj.data = _.assign({}, obj.data, {name: 'a'}); //或者这样
    ```
    如果需要修改，建议给直接子属性赋值新对象。
## 执行多次 _.times  
有时候就需要一个简单的数组，或者简单重复的工作，这时候写for有点繁琐，这个函数提供了简便的方法
```
_.times(3, Number)// [0, 1, 2]
_.times(3, String)// ['0', '1', '2']
_.times(3, function() {return 1;});// [1, 1, 1]
```
## 数组转对象：_.keyBy  
当得到一个数组，要通过这个数组内对象的id或者别的属性找到这个对象时，比起遍历匹配，这样做更高效
```
var obj = _.keyBy(list, 'id');
// 查找时只需要
var item = obj[id];
```
## 是否包含：_.includes  
是否包含，对象数组字符串都可以用，比起用indexOf与-1作对比，语义更好。另外ie8中Array不支持indexOf。
```
_.includes([1, 2, 3], 1);// => true
_.includes([1, 2, 3], 1, 2);// => false
_.includes({ 'a': 1, 'b': 2 }, 1);// => true
_.includes('abcd', 'bc');// => true
```
## 获取最大：_.maxBy()   
在我的一个项目中，我需要找出一堆坐标中，找出最大的y是多少，这时我可以：
```
var item = _.maxBy(list, function(o){return o.y;}); //返回数组中y最大的对象
// {x: 0, y: 3}
```

## 小结  
lodash是一个对数组、对象进行操作很方便的库，性能也很好，这也是他能被这么多npm包依赖的原因。是我知道的一部分，lodash肯定还有很多好用的方法我不知道，待以后慢慢发现。