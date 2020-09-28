## 网页渲染原理

![domtree](/imgs/domtree.webp)

  



这部分工作主要是渲染进程完成  



1.HTML代码转化成DOM  

2.CSS代码转化成CSSOM（CSS Object Model）  

 3.结合DOM和CSSOM，生成一棵渲染树（包含每个节点的视觉信息）  

 4.生成布局（layout），即将所有渲染树的所有节点进行平面合成  

 5.将布局绘制（paint）在屏幕上  

  

![rendertree](/imgs/render-tree.webp)