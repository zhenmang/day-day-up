## mvc

![](/imgs/mvc.webp)

优点：
1）职责分离思想，MVC三者各司其职，模块化；
2）观察者模式，实现单个Model可更新多视图更新。

缺点：View依赖Model，不可避免引入业务逻辑，不易复用。

## mvp

![](/imgs/mvp.webp)

优点：解决View与Model耦合问题，使View变薄，更易复用。

缺点：Presenter 承担了V->M和M->V的应用和业务逻辑，容易变得臃肿，可维护性降低。

## mvvm

![](/imgs/mvvm.png)

亮点：View 和 ViewModel 间的数据同步。

