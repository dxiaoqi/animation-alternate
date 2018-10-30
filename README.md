# animation-alternate
动画与交互实现(前端)
[TOC]



## 1. 空间与转换

当图形被绘制在屏幕上的时候，无论是2D还是3D，都会有其自己的空间，也会有其自己的转换数据。

### 空间坐标

1. 齐次坐标和转换矩阵：
  在计算机图形学中，通常是才用齐次坐标来表示空间内的点,在三维空间内，会使用四元向量来表示。

  ​						$$[\frac{x}{w},\frac{y}{w},\frac{z}{w}]=[x,y,z,w]$$

一般w的默认值为1，较为基本的旋转，平移，缩放多采用的是4维矩阵，当我们需要一些复杂的操作时，还可以通过矩阵获得复合矩阵。



### 基本的转换操作

无论是css还是canvas等图形的转换操作，采用的操作都是相同的。
**平移**

$$\begin{matrix}x\\y\\z\\1\end{matrix}=\begin{matrix}x\\y\\z\\1\end{matrix}*\begin{matrix}1&0&0&t\\0&1&0&t\\0&0&1&t\\0&0&0&1\end{matrix}$$

**旋转x**

$$\begin{matrix}x\\y\\z\\1\end{matrix}=\begin{matrix}x\\y\\z\\1\end{matrix}*\begin{matrix}1&0&0&0\\0&cosX&-sinX&0\\0&sinX&cosX&0\\0&0&0&1\end{matrix}$$

**旋转y**

$$\begin{matrix}x\\y\\z\\1\end{matrix}=\begin{matrix}x\\y\\z\\1\end{matrix}*\begin{matrix}cosX&0&sinX&0\\0&1&0&0\\-sinX&0&cosX&0\\0&0&0&1\end{matrix}$$

**旋转z**

$$\begin{matrix}x\\y\\z\\1\end{matrix}=\begin{matrix}x\\y\\z\\1\end{matrix}*\begin{matrix}cosX&-sinX&0&0\\sinX&cosX&0&0\\0&0&1&0\\0&0&0&1\end{matrix}$$

**放缩**

$$\begin{matrix}S1x\\S1y\\S1z\\1\end{matrix}=\begin{matrix}x\\y\\z\\1\end{matrix}*\begin{matrix}S1&0&0&0\\0&S1&0&0\\0&0&S1&0\\0&0&0&1\end{matrix}$$

也许在看3D的图形转换的时候，会感觉好复杂，但是当我们去看2D的时候，砍去了一个维度，公式也就固定了。

$$\begin{matrix}x\\y\\1\end{matrix}=\begin{matrix}x\\y\\1\end{matrix}*\begin{matrix}
cosX&-sinX&0\\sinX&cosX&0\\0&0&1\end{matrix}$$

进一步简化

$$\begin{matrix}x\\y\\\end{matrix}=\begin{matrix}x\\y\\\end{matrix}*\begin{matrix}cosX&-sinX\\sinX&cosX\\\end{matrix}$$

这个时候，我们会得到一段较为常见的旋转代码
```js
/**
向量定义
var Vector2={
    x:0,
    y:0
}
**/
function rotate(site,angle=0){
    	var _angle=angle/180*Math.PI;//将弧度转换为角度
    	//进行计算
    	var x1=site.x*Math.cos(_angle)-site.y*Math.sin(_angle);
    	var y1=site.x*Math.sin(_angle)+site.y*Math.cos(_angle);
    	//返回新的向量
    	return {
    	    x:x1,
    	    y:y1
    	}
};
```
**局限性:** 矩阵的数据转换因为数据格式化，所以并不适用于如非线性动画的转换。 
>一周是360度，也是2π弧度。弧度是这样定义的，一个角对应的弧长与半径的比值就是弧度。半径为1的圆周长是2π，所以360度=2π弧度，以后的类推就行了。几个重要的角度还有：30度=π/6弧度，60度=π/3弧度，90度=π/2弧度，180度=π弧度等。

### 向量之说

在空间之中，可以被划分为空间坐标与对象坐标
用CSS来表示的话，空间坐标有些类似`position:absolute`以整个视图的原点为基准。而对象坐标的说法更贴切的应该是相对坐标，类似`position:relative`
为了方便对于坐标进行计算以及数据转换，空间中的任何点信息都可以使用向量来作为信息载体。

**Example:**
(1,1)可以表示为空间中x=1,y=1的坐标点，也可以表示为从(0，0)到(1，1)的距离。
重新定义一个Vector2的类
```js
function Vector2(x=0,y=0){
	if(!(this instanceof Vector2)){
		return new Vector2(x,y);
	}
	this.x=x;
	this.y=y;
}
Vector2.prototype = {
    copy: function() {//返回新的向量
    	 return new Vector2(this.x, this.y); },

    length: function() {//当前向量的长度
    	 return Math.sqrt(this.x * this.x + this.y * this.y); },

    normalize: function() {//单位向量
     var inv = 1 / this.length(); 
     return new Vector2(this.x * inv, this.y * inv); },

    negate: function() {//反向向量 
    	return new Vector2(-this.x, -this.y); },

    add: function(v) {//向量和
    	return new Vector2(this.x + v.x, this.y + v.y); },

    subtract: function(v) {//向量差
     return new Vector2(this.x - v.x, this.y - v.y); },

    multiply: function(f) {//向量积 
    	return new Vector2(this.x * f, this.y * f); },

    divide: function(f) { //向量方向化
    	var invf = 1 / f; 
    	return new Vector2(this.x * invf, this.y * invf); },

    dot: function(v) {//点积 
    	return this.x * v.x + this.y * v.y; },
    move:function(v){
        this.x=v.x;
        this.y=v.y;
        return this;
    },
    prependicular:function() {//法向量
    	return new Vector(this.y, -this.x);
	}，
    rotate:function(angle=0){
    	var _angle=angle/180*Math.PI;
    	this.x1=this.x*Math.cos(_angle)-this.y*Math.sin(_angle);
    	this.y1=this.x*Math.sin(_angle)+this.y*Math.cos(_angle);
    },
};
```
向量的运用：速度(v),力(f),方向(d)，颜色(rgb)等...

当我们把信息使用向量存储值后，就会发现很多功能都是清晰明了，比如属性的插值运算



### 角度
角度的计算，在计算机动画实现中，有**定角表达** **欧拉角表达** **轴角表达**这三种说法，不过这些都不需要去了解，因为在插值计算的过程中，这些技术并不合适，如果想深入了解原因的，可以去了解一下什么是**万向节死锁(gimbal lock )**。

#### 欧拉角

欧拉角是表达旋转最简单的一种方式，表达了物体绕坐标系的轴的旋转角度，2D平面内提供了大量的旋转api ,css里的`transform:rotate(90deg)`，canvas里的`ctx.rotate(angle)`，对于3D方面,css也是提供了在各个轴向上的Rotate，canvas则更多是在webgl中使用的矩阵变换。

对于欧拉角的定义，有人概括了一下几点。

1. 旋转角的组合方式:以(x,y,z)来说明就是角度的执行顺序，如X-Y-Z或者Z-X-Y，用css来说就是`X-Y-Z== rotateX()-rotateY()-rotateZ()`
2. 旋转角度的参考坐标系统（旋转是相对于固定的坐标系还是相对于自身的坐标系）
3. 使用旋转角度是左手系还是右手系

#### 万向节死锁

在欧拉角中，我们可以发现，在轴转向的时候，会有一个顺序，如果当角度不恰当，会导致轴旋转的过程中，有两个轴会发生重合，导致维度降低。

![](C:\Users\Administrator\Desktop\canvas\a\glockgif.gif)

当然，我们也可以使用代码来对万向节死锁进行复现。

```js
Point.Rotate(new Vector3(0, 0, 10));  
Point.Rotate(new Vector3(0, 90, 0));  
Point.Rotate(new Vector3(20, 0, 0));  
```

只需要固定住某一个轴的转角为90°，无论怎么去调整其他的轴，都会发现，他们只会在平面上运动。

我们所要了解的是 **四元数**,这个词的概念在游戏开发中很常见。那么选择四元数来处理自由度旋转的优势在哪里呢。
**优势**

1. 不存在万向节死锁
2. 计算效率高(矩阵旋转效率较低)
3. 可以以物体的中心点为轴来做旋转

**弱点**
1. 旋转轴限制(矩阵旋转可以任意轴)
2. 不可以超过180°(矩阵旋转无限制)

在了解四元数之前，我们要了解一个知识点**复数**，如果已有基础，可以跳过。
#### 复数
**定义:**
任意一个复数 z ∈ C 都可以表示为 z = a +bi的形式，其中 a, b ∈ R 而且 $$i^2 = −1$$．我们将 a 称之为这个复数的实部（Real Part），b称之为这个复数的虚部（Imaginary Part）．
如果将复数使用坐标系来表示。
![](C:\Users\Administrator\Desktop\canvas\a\fushu.jpg)

#### 四元数

四元数是一个恐怖的东西，因为当把他放在图形中去理解，你会发现比矩阵的还要难理解很多，在正常的坐标系中，每个轴都会是一个直线，而在四元数中，多出一个轴向，而且这个轴会垂直于任何一个轴，相对于复数的二维空间，四元数则是三维的复数形式，是一种高阶复数，感觉像就是四维空间。

四元数的数学表达还是比较好理解的$$Q=w+xi+yj+zk$$,Q是一个四元数，w是一个实部，x,y,z则是虚部，且$$i^2+j^2+k^2=-1$$。

当四元数应用到旋转中的时候，我们通常可以这么表示一个$$Q=(w,(x,y,z))=(w,v)$$,w是实数，v是向量，每一次的旋转都会需要两个四元数来配合，四元数的的范围在[-1,1]之间。

接下来我们试着实现一个四元数

```js
/* 四元数
*/
 
class Quaternion{
    constructor(x=0,y=0,z=0,w=0){
        this.x=x;
        this.y=y;
        this.z=z;
        this.w=w;
    }
    fromAxisVector(axisVector,angle){// 由 旋转轴向量，旋转角 得到
        var t = sin(0.5*angle);
 		this.w = cos(0.5*angle);
		this.x = axisVector.x * t;
		this.y = axisVector.y * t;
		this.z = axisVector.z * t;       
    }
    add(q){
  		this.w += q.w;
		this.x += q.x;
		this.y += q.y;
		this.z += q.z;     
    }
    subtract(q){
   		this.w -= q.w;
		this.x -= q.x;
		this.y -= q.y;
		this.z -= q.z;          
    }
    multiply(q){
        var {x,y,z,w}=q;
 		this.w = w*q.w - x*q.x - y*q.y - z*q.z;
		this.x = w*q.x + x*q.w + y*q.z - z*q.y;
		this.y = w*q.y + y*q.w + z*q.x - x*q.z;
		this.z = w*q.z + z*q.w + x*q.y - y*q.x;       
    }
    normalize(){
        var {x,y,z,w}=this;
 		var magnitude = Math.sqrt(x*x + y*y + z*z + w*w);
		if (magnitude != 0)
		{
			x /= magnitude;
			y /= magnitude;
			z /= magnitude;
			w /= magnitude;
		}       
    }
    convertToMatrix4(){//转换为矩阵
 		//  四元数与矩阵的转换
		//     [ 1-2y2-2z2 , 2xy-2wz , 2xz+2wy ]
		//     [ 2xy+2wz , 1-2x2-2z2 , 2yz-2wx ]
		//     [ 2xz-2wy , 2yz+2wx , 1-2x2-2y2 ]
 		var {x,y,z,w}=this;
		var xx = x*x;  var xy = x*y; 
        var xz = x*z;  var xw = x*w; 
		var yy = y*y;  var yz = y*z;  
        var yw = y*w;  var zz = z*z;  var zw = z*w;
 
		return Matrix4(  1-2*(yy+zz),  2*(xy-zw),    2*(xz+yw),    0,
							2*(xy+zw),    1-2*(xx+zz),  2*(yz-xw),    0,
							2*(xz-yw),    2*(yz+xw),    1-2*(xx+yy),  0,
							0,            0,            0,            1  );
       
    }
}
 

```



## 2. 插值计算

插值运动是指通过一些离散的数据进行数据的拟合，从而推断出新的未知数据点，使用简单函数来模拟复杂函数，从而提升数据的精度。

插值计算在运动之中，最常见的就是属性插值，如颜色渐变,宽高过度,缓动动画等，主要是通过计算机自行去计算，实现自动补帧。Flash中的补间动画采用的就是插值补间补帧。

假设给定n个离散数据,定义了其坐标为$$(x_k,y_k),k=1,2,3...$$ 在区间 $$[a,b]$$ 上有函数g(x), 可以满足$$g(x_i)=f(x_i)$$,那么g(x)则可以被称为是f(x)在的  [a,b] 上插值函数，这也就是使用简单函数来模拟复杂函数。

| 属性        | 插值类型 | 效果                  |
| ----------- | -------- | --------------------- |
| color/alpha | 线性     | (颜色/透明度)渐变过度 |
| 加速度      | 线性     | 匀变速                |
| 欧拉角      | 线性     | 旋转                  |
| 速度        | 非线性   | 变加速                |



### 线性插值

线性插是一种很常见的插值方法，在动画计算中很常见，可以用来实现自动补帧，其基本的实现也较为简单。

![](C:\Users\Administrator\Desktop\canvas\a\Linear_interpolation.png)

线性插值一般是采用两点数据进行计算，最常见的就是直线插值，tween.js的Linear就是线性插值的一个实例。

```js
/*
 * t: current time（当前时间）；
 * b: beginning value（初始值）；
 * c: change in value（变化量）；
 * d: duration（持续时间）。
*/    
Linear: function(t, b, c, d) { 
        return c * t / d + b; 
    }
```



### 多项式插值

多项式插值是线性插值的一个延伸，在线性插值的原公式上，支持了高阶多项式计算。

```js
    Quad: {
        easeIn: function(t, b, c, d) {
            return c * (t /= d) * t + b;
        },
        easeOut: function(t, b, c, d) {
            return -c *(t /= d)*(t-2) + b;
        },
        easeInOut: function(t, b, c, d) {
            if ((t /= d / 2) < 1) return c / 2 * t * t + b;
            return -c / 2 * ((--t) * (t-2) - 1) + b;
        }
    }
```

这是`Tween.js`中的二次方插值，同时，还包含了三次方插值，甚至五次方插值。

### 三角插值

三角插值这里指的就是三角函数COS TAN SIN，以x轴与y轴形成关系.如:

- v=_v*Sin(t) 速度随着时间的增长而产生变化



## 3. 基本动画

有了之前的基础知识与插值的基础，就有了足够的而基础去进行动画的尝试。

于是我们可以从一个点开始构建

```js
class Point{
    constructor(x,y){
        this.pos=new Vector2(x,y);
    }
    draw(){
        //图形绘制
    }
    updata(){
        //逻辑处理，数据更新
    }
}
```

这里的点已经具有了`Vector2`的方法，从而使得这个点在二维空间中具有了一定的能力，包括平移，旋转。

之前有说，几乎所有的属性都可以使用向量作为载体，于是这里，可以使用`Vector2`给`Point`赋予很多的属性，便可以得到

```js
class Point{
    constructor(x,y){
        this.pos=new Vector2(x,y);
        this.f=new Vector(0,0);
        this.m=10;
        this.a=this.f.length()/this.m;
    }
}
```

很简单的一个$$F=m*a$$ 公式，就给`Point`赋予了接受外界力的能力，以及运动的能力。

$$F=m*a$$  (F为合力,F是个矢量) 

$$v= v_0+at$$   

$$S=v_0t+\frac{at^2}{2}$$ 

这几个公式是力与运动学之中最常用也是最关键的几个公式，也是运动学中很关键的一步，那么如何正确的去计算一个物体的运动状态呢。

1. 判断物体当前状态，是单体，还是有链接状态
2. 对物体所受力进行求和，对单个`Point`进行`updata`
3. 对物体进行重绘

这样，就可以将基本运动的动画利用物理公式从而实现，如匀加速，变加速，圆周运动等。

## 4.动画中的状态机

### 链式动画

状态机在游戏开发中是一个很常见的词汇，那么状态机的存在是为了什么，在哪些地方有运用呢，

首先以`Point`为基础，添加一个状态量

```js
const PEDDING='PEDDING';//静止状态
const MOVING ='MOVING';//运动状态
const SHOW   ='SHOW';//显示
const OUT    ='OUT';//屏幕之外

//状态判断
if(Point.status=='PEDDING'){
    cb();
}
```

这么看起来是不是有些熟悉，对比发现，`promise`其实也是一个状态机，不断判断当前的执行状态，来确定何时进行下一个事件的执行，对比着`promise`的链式调用，也就可以轻易的去明白一些动画库中的链式调用原理。

### 资源管理器

在视图中进行动画的物体，总会有一部分会消失在视图之外，为了降低了内存占有，也许可以直接使`obj=nul`，但是当我们仍需要其后续的出现，再去使用申请一个新的对象？显然有很多不合理的地方，于是便有了资源管理器。

```js
var p1=new Point(0,0);
var p2=new Point(1,1)
var resource=[p1,p2];
//状态判断
resource.forEach(p=>{
    if(p.status=='SHOW'){
        p.updata();
        p.draw();
    }
    if(p.status=='OUT'){
       	//对p进行移除或者重置设置
    }
})
```

这样的优势是可以降低大量的计算以及渲染工作，如果打算彻底移除某个物体，则可以使用`Array.splice`

### 用户交互

用户交互也是很常用的一个状态机，以canvas为例，用户的事件监听是针对canvas整体的，如果我们想实现一个拖拽的功能。

状态分析：

1. 正常情况，鼠标释放，status·为UP
2. 按下的状态，status为DOWN
3. 按下后移动鼠标，status为DROP

状态机的存在是以鼠标事件为本体。

## 5.碰撞检测

实现了物体基本的运动与交互，那么接下来需要实现的就是物体与物体的交互，现在在我们所了解到的碰撞检测方法。

1. 包围盒
2. 包围球

这两个也是最为简单计算，也是最适合做粗计算阶段的碰撞检测，可以将一些不必要进行进行精密计算的物体图形排除在外，减少计算量

### 包围盒

以物体中心为基础，生成最小的包围矩形

```javascript
rectB.x > rectA.x - rectB.width &&
rectB.x < rectA.x + rectA.width + rectB.width &&
rectB.y > rectA.y - rectB.height &&
rectB.y < rectA.y + rectA.height + rectB.height
```

### 包围球

以物体为基础，生成最小的包围球形

```javascript
Math.sqrt(Math.pow(circleA.x - circleB.x, 2) + Math.pow(circleA.y - circleB.y, 2)) < circleA.radius + circleB.radius
```

### 分离轴

分离轴也许听起来晕，甚至看网上的一些讲解也很晕，那么可以考虑在这个时候打开网易云音乐，点一首你最爱的歌，然后开始阅读。

分离轴，顾名思义是将轴分离开，那么在我们所了解的领域中，最长出现的就是x轴与y轴，这也是坐标系的基础，那么轴的特点是什么，**垂直**，这也是分离轴的依据所在。

分离轴的实现有些像模拟灯光投影，当光线穿过两个空间中的物体，为了防止影子变形，设置一个垂直光线的挡板，想像一下，如果光线可以从两个物体中穿出，那么两个物体之间就不存在接触，那么投射的影子也就不会出现重叠，当足够多的光线进行穿透，如果出现垂直光线的挡板没有出现阴影重叠，那么我们就可以认定这两个物体没有发生碰撞。

> 碰撞的检测，是只需要一组轴的检测未重合，那么可以判定为分离，如果所有轴的检测都重合，则物体发生碰撞

于是这里我们就有了两个轴，光轴与投影轴。

![](C:\Users\Administrator\Desktop\canvas\a\shadown.jpg)

于是我们有了第一缕阳光

```js
var Light=new Vector2(0,0);
```

让阳光来穿过物体

```js
var Point1 =new Point(0,0);
var Point2 =new Point(0,1);
var Light =Point1.pos.subtract(Point2.pos);//光线向量
var Panel =Light.prependicular();			//获取投影轴的向量
var axis  =Panel.normalize();				//轴的单位向量,为投影点做准备

```

求出我们的投影点，这里所需要的公式  $$v_1*v_2=|v_1|*|v_2|*conθ$$

```javascript
Light.dot(axis);
```

得到了投影点后，一个物体在一个轴面上的投影点的最大值与最小值的差值，就是阴影面的范围。

### 像素检测

像素检测的方法就是将每个物体当前的像素位置都存储起来，再比较物体之间的像素是否有重复，但是计算量庞大。

### 检测优化

#### 栅格化

栅格化的意思就是将屏幕划分为数个小块，对不同区域内的物体进行单独处理，对于处于分界线上的物体，则可以进行多次判断。最常用的栅格法就是**四叉树**

![](C:\Users\Administrator\Desktop\canvas\a\4-tree.png)

## 6.物理动画

因为现实中有很多感染因素，所以说物理动画的实现是对真实运动的有限模拟。

### 基本物理动画

### 链接动画(约束动力学)

### 物理建模基础
