### Perspective

属性作用：透视，可以近似理解为视图与人眼的距离。

滑动开关，调整与人眼的距离。

将以下代码复制到浏览器环境中执行。

{% urlembed %}
/iframes/perspective.html
{% endurlembed %}

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<style>
.control {
  text-align: center;
}
.stage {
  margin:auto;
  perspective: 200px;
  position: relative;
  width:400px;
}
.actor {
  /* position: absolute; */
  width: 400px;
  height: 400px;
  background-color: rgba(255, 100, 100, 0.5);
}
</style>
<body>
  <div class="control">
      <input type="range" onchange="fn(value)" max="250" min="0" value="0">
  </div>
  <div class="stage">
    <div class="actor"></div>
  </div>
  <script>
    let divStyle = document.getElementsByClassName('actor')[0].style
    function fn (value) {
      divStyle.transform = 'translateZ(' + value + 'px)'
    }
  </script>
</body>
</html>
```