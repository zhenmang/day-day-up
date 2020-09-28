## 性能优化思考方向：

1. 资源及代码压缩  

   

2. 合理的减少请求  

   

3. 异步加载阻塞资源  

   

4. 减少回流和重绘  

   

## To Do List：

1. 精简DOM结构，减少布局计算量  

   

2. 少操作dom，尽量使用documentFragment再appendChild或者cloneNode再replace  

   

3. js读写dom属性时，采用读写分离，避免强制回流  

   

4. 减少对象的遍历深度和遍历次数  

   

5. 用IIFE包裹所有js，强化内部的垃圾回收  

   

6. 提高css选择器的精准度  

   

7. GPU加速 transform:translate3d(0, 0, 0)  

   

8. 动画脱离标准文档流，少用定时器、延时器，多用requestAnimationFrame  

   

9. 坚持使用 transform 和 opacity 属性更改来实现动画，只触发合成器，不触发回流和重绘  

   

## 结语

这仅仅是从浏览器端的优化方向，考虑前后端，用户的初屏速度、资源的传导速度，还有一些前后端可以一起做的优化点。

