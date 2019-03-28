### **Transform-style: preserve-3d**

属性作用：保存子元素的3D效果。

旋转的立方体。

将以下代码复制到浏览器环境中执行。
{% urlembed %}
/iframes/transform-style.html
{% endurlembed %}

```HTML
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
        <style>
            .container{
                width: 500px;
                margin: 50px auto;
            }
            .cub{
                position: relative;
                width: 100px;
                height: 100px;
                transform-style: preserve-3d;
                -webkit-transform-style: preserve-3d;
                transition: all 5s linear;
            }
            .cub div{
                width: 100px;
                height: 100px;
                position: absolute;
                left: 0;
                top: 0;
                transform-style: preserve-3d;
                -webkit-transform-style: preserve-3d;
                border: 1px solid #46B8DA;
                opacity: 0.5;
            }
            .one{
                transform: rotateY(0deg) translateZ(50px);
            }
            .two{
                transform: rotateY(180deg) translateZ(50px);
            }
            .three{
                transform: rotateX(90deg) translateZ(50px);
            }
            .four{
                transform: rotateX(-90deg) translateZ(50px);
            }
            .five{
                transform: rotateY(90deg) translateZ(50px);
            }
            .six{
                transform: rotateY(-90deg) translateZ(50px);
            }
            .cub:hover{
                transform: rotateX(360deg) rotateY(360deg);
                -webkit-transform: rotateX(360deg) rotateY(360deg);
            }

            /* .cub:hover .one{
                transform: rotateY(0deg) translateZ(100px);
            }
            .cub:hover .two{
                transform: rotateY(180deg) translateZ(100px);
            }
            .cub:hover .three{
                transform: rotateX(90deg) translateZ(100px);
            }
            .cub:hover .four{
                transform: rotateX(-90deg) translateZ(100px);
            }
            .cub:hover .five{
                transform: rotateY(90deg) translateZ(100px);
            }
            .cub:hover .six{
                transform: rotateY(-90deg) translateZ(100px);
            } */
        </style>
    </head>
    <body>
        <div class="container">
            <div class="cub">
                <div class="one" style="background-color: #FFC0CB"></div>
                <div class="two" style="background-color: #449D44"></div>
                <div class="three" style="background-color:#46B8DA"></div>
                <div class="four" style="background-color: #9E90EE"></div>
                <div class="five" style="background-color: #C0A16B"></div>
                <div class="six" style="background-color: #D6E9C6"></div>
            </div>
        </div>
    </body>
</html>

```