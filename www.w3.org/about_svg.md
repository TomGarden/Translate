原文：https://www.w3.org/TR/SVG/paths.html

## 9.3.6. The cubic Bézier curve commands

*name*      :   **curveto**
- *command*   :   `C` (absolute)   `c` (relative)
- *parameter* :   (x1 y1 x2 y2 x y)+
- *descript*  :   从当前点到 (x,y) 绘制一条三次 Bézier 曲线，(x1,y1) 作为曲线始点的控制点，(x2,y2) 作为曲线终点的控制点。C(大写)从绝对位置开始绘制；c(小写)从相对位置开始绘制。多次设置坐标可以绘制一个聚合 Bézier 曲线。在绘制聚合 Bézier 曲线命令结束的时候当前坐标就变成了 (x,y) 。

*name*          :   **shorthand/smooth curverto**
- *command*       :   S(absolute)   s(relative)
- *parameters*    :   (x2 y2 x y)+
- *description*   :   从当前点 (x,y) 开始绘制一条三次 Bézier 曲线。第一个控制点被假设为前一个命令中第二个控制点相对于当前点的反射。(如果不存在‘前一个命令’或者‘前一个命令不是 C/c/S/s ,就假设第一个控制点和当前点重合‘。)(x2,y2) 是第二个控制点(曲线末端的控制点)。 S(大写)从绝对位置开始绘制；s(小写)从相对位置开始绘制。多次设置坐标可以绘制一个聚合 Bézier 曲线。在绘制聚合 Bézier 曲线命令结束的时候当前坐标就变成了 (x,y) 。