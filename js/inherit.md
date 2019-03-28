继承的实现有很多种，以下是个人感觉最棒的两种

**一、原型链继承**

```
function People () {
  this.kind = 'people';
}
People.prototype.eat = function() {
};

function Man(name){
	this.name = name
}
Man.prototype = new People();
```

分析：Man类原型指向了People类的实例

优点：继承了People类上的属性及原型方法

缺点：可能People类上的属性并不是我们想要继承的



**二、寄生组合继承**

```
function Man(name){
  People.call(this);
  this.name = name;
}
(function(){
  // 创建不带属性的中间类
  var Mid = function(){};
  // 中间类的原型指向父类的原型
  Mid.prototype = People.prototype;
  // 子类的原型指向中间类的实例
  Man.prototype = new Mid();
})();
```

分析：Man类通过无属性的中间类继承了People类

优点：Man类只继承了People类原型上的方法，而没继承People类上的属性

缺点：实现过程中，多执行了中间类