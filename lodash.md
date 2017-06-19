## 分享lodash中好用的方法

# _.chain()  
感觉像gulp的管道。打一个包，然后可以链式调用，当需要对数组做一系列操作时可以用。  
chain是延迟执行的，只有调用value方法，才会真正计算，所以性能很好。
```
// 我的需求渲染一个表格，取sort前三个，然后data1大于1显示红色，小于1显示绿色
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
# _.get()  
按照路径获取数据，可以设置默认值。
```
_.get(obj, 'a.b.c.d', null);
```
当后台返回的数据结构比较深，而且某个层级的数据不确定有没有时，可以使用这个方法。
例子：
```
var data = res.data.comnany.legalPerson.name;

// 如果legalPerson不存在，这个代码怎么写都会报错
// 'Cannot read property 'name' of undefined'，所以我们会这样写

var name = '';
var legalPerson = res.data.comnany.legalPerson;
if (legalPerson) {
    name = legalPerson;
}

// 假如连company都不确定呢，那要写的if就会很长了
// 当然如果你相信你们的后台100%会给你完整的数据结构，那就没必要用这个方法
// 但是以我的经验...，还是用这个比较保险

var name = _.get(res, 'data.company.legalPerson.name', '');
```
# _.assign()  
可用于复制、合并对象，不是深拷贝，只拷贝一层。
```
_.assign({}, a, b); // 经常这样用
```
在写类的构造函数时，经常会有这种场景：我有一些默认配置属性，在使用的时候还允许传入新的配置属性，来覆盖默认配置属性。如果我们没有其他库，可能需要写个遍历：
```
for (var i in config) {
    if (default.hasOwnproperty(i)) {
        default[i] = config[i];
    }
}
// 这个时候，我们也可以用assign
default = _.assign(default, config);
```
1. 关于类似的方法  
jquery中：$.extend(default, config);  
ES6中：Object.assign(default, config);

2. 与_.assign的异同  
在lodash3版本，以上方法与_.assign特性相同。  
在ladash4版本，以上两种方法基本上等同于_.assignIn，新版的_.assign不会改变第一个对象的值。  
3. 关于使用时，为什么经常第一个参数写一个‘{}’
在使用lodash3版本_.assign、$.extend()和Object.assign时，需要源数据也会被修改，这个会造成污染，没办法获取到源数据，所以一般都这样用：
```
var obj = _.assign({}, default, config);
```
4. 关于浅拷贝与深拷贝
assign方法只会拷贝一层，建议如果修改，就直接给子属性赋值新对象，给更深层次的属性赋值会有引用
```
var obj = _.assign({}, default, config);
obj.data.name = 'a'; // 这样config或default里的数据也会被修改
obj.data = {name: 'a'}; //这样才不会修改其他的值
obj.data = _.assign({}, obj.data, {name: 'a'}); //或者这样
```
lodash还有一个merge方法，这个可以实现子属性的合并，而不是覆盖。
# _.times()  
执行多次。有时候就需要一个简单的数组，或者简单重复的工作，这时候写for有点繁琐，这个函数提供了简便的方法
```
_.times(3, Number)// [0, 1, 2]
_.times(3, String)// ['0', '1', '2']
_.times(3, function() {return 1;});// [1, 1, 1]
```
# _.keyBy()  
数组转为对象，设置一个属性为key
```
_.keyBy(list, 'a'); //返回对象以a的值为key
```
当得到一个数组，要通过这个数组内对象的id或者别的属性找到这个对象时，比起遍历匹配，这样做更高效
```
var obj = _.keyBy(list, 'id');
// 查找时只需要
var item = obj[id];
```
# _.includes()  
是否包含，对象数组字符串都可以用，比起用indexOf与-1作对比，语义更好。另外ie8中Array不支持indexOf。
```
_.includes([1, 2, 3], 1);// => true
_.includes([1, 2, 3], 1, 2);// => false
_.includes({ 'a': 1, 'b': 2 }, 1);// => true
_.includes('abcd', 'bc');// => true
```
# _.now()  
现在的时间戳
```
_.now(); //1497509692736
```
# _.maxBy()   
找最大
```
_.maxBy(list, function(o){return o.a;}); //返回数组中a最大的
```

## 这是我知道的一部分，lodash肯定还有很多好用的方法我不知道，待以后慢慢发现。