+++
resources = []
title = "GIS 计算中的凸包、凹包、内凸包等问题求解"
mainContent = ""
pubdate = "2022-02-17"
categories = ["GIS"]
featuredImage = "/covers/9.jpeg"
+++

## 1. 最小凸包 convex hull

### 1.1 凸包的定义（convex）


- 几何定义：
    - 如果一个面是 convex，那它边上的任意两点相连出来的线段都落在面的内部。
    - 如果一个面是 convex，那它内部任意两点相连都将面划分为半空间 [half-space](https://en.wikipedia.org/wiki/Half-space_(geometry))
- 表象：所有两个边所成的内边角不大于180度
- 通俗的表象：将点的集合看成一根根柱子，用一根皮筋圈套住所有的柱子形成的皮筋形状就是凸包的形状。

<p>
<image src="/convex-hull/1.png" style="width:400px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

<image src="/convex-hull/2.png" style="width:400px;border-width:3px; border-color:#666;border-style:solid;"/>


### 1.2 最小凸包的定义 convex hull

针对一群点的集合，其最小面积的包含所有点的凸集合形成的面

<image src="/convex-hull/3.png" style="width:400px;border-width:3px; border-color:#666;border-style:solid;"/>

### 1.3 特性或用途

- 凸包是一个确定的、包容的、平滑的、优雅的图形。
- 实际上凹包用的更多。。。
### 1.4 凸包计算步骤（滚球法）

<image src="/convex-hull/4.png" style="width:400px;border-width:3px; border-color:#666;border-style:solid;"/>

- 求极心 P0
- 找出屏幕最左边的一个点，这个点一定是凸包边的一部分
- 选中 P1，寻找下一个点 P2
    - 求极心到上一个选中的点的角度 P0->P1，目的是作为内外侧判断的基准
    - 用已选定的点轮训所有的点，求角度 P1->P2，然后求P1->P2与P0->P1的角度差
    - 选中角度差最小的点 P2
    - 使用 P2 循环以上过程


<image src="/convex-hull/8.png" style="width:700px; border-width:3px; border-color:#666;border-style:solid;"/>

<image src="/convex-hull/9.png" style="width:700px; border-width:3px; border-color:#666;border-style:solid;"/>

演算视频：

<video src="/convex-hull/10.mp4" style="width:600px; border-width:3px; border-color:#666;border-style:solid;" controls autoplay muted />


## 2. 凹包 concave hull

### 2.1 凹包定义

- 没有明确的定义，也没有唯一值
- 可以这样定义：Concave Hull 函数创建一个多边形，该多边形表示一组数据点所占据的区域。

<image src="/convex-hull/7.png" style="width:500px;border-width:3px; border-color:#666;border-style:solid;"/>

<p>
<image src="/convex-hull/11.png" style="width:500px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>
<p>
<image src="/convex-hull/12.png" style="width:500px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

### 2.2 凹包用途

凹包虽然是一个没有唯一值和明确定义的图形，但是它实际上比凸包使用场景甚至更广泛。因为他代表的是尽量真实的一组点所占据的区域。

例如：现有一个较为密集的记录时间、位置的用户行驶轨迹数据，获取尽量准确的用户活动发生区域

回到这张图，也可以非常准确的表达这个需求，在这个场景下，凸包实际上对需求的表达不是很准确，而尽量逼近的凹包才是我们想要的。

<p>
<image src="/convex-hull/13.png" style="width:500px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

又例如我之前做过的一个功能，在纵横交错的N条不规则的水平和垂直道路中，求任意四条的合围，把横着的路变成面，竖着的路变成面，最终取交集，这里怎么把很多条很复杂的线变成面，就需要用到凹包，凸包在这里完全发挥不了作用。

<p>
<image src="/convex-hull/14.png" style="width:500px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

### 2.3  凹包计算步骤（滚球法）

<p>
<image src="/convex-hull/5.gif" style="width:500px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

<image src="/convex-hull/6.png" style="width:500px;border-width:3px; border-color:#666;border-style:solid;"/>

- 求极心 P0
- 找出屏幕最左边的一个点，这个点作为凹包边的一部分 P1
- 选中 P1，进入寻找 P2 的逻辑
    - 滚球法使用的球的半径，实际上代表的是一个扫描范围
    - 所以，下一步以 P1 为圆心，过滤出来固定半径范围内的点：nearestPoints。
    - 遍历 nearestPoints，以 P0->P1 角度为基准，逆时针旋转寻找最近的点。
        - 假设当前遍历到 P2
        - 计算 P1->P2 的角度，然后比对 P0->P1和 P1->P2 在逆时针上最接近的 P2
    - 最终遍历结束后，逆时针角度最小的点就是要寻找的 P2
    - 一直执行，直到回到起点或者执行完了一周的遍历


注意，凹包的特性如下：
- 凹包不是唯一的。
- 调节滚球的半径，可以获得不同的结果，什么时候用什么样的半径？不太好确定。
- 如果点一会比较集中一会比较离散，很容易断线，也就是在一个 P1 中，球滚了一圈还没有结果，这时候要么提高滚球半径，要么就会得到一个不完全的凹包。
- 提高滚球半径，会导致密集区域无法紧贴集合的边缘，二者矛盾。
        

### 2.4. 改进凹包计算（自动伸缩滚球法）

对于密集程度比较平均的集合，上述方法还算有效，但是当你拿到的是一个密集程度不均匀的集合的时候，你会发现这种普通滚球基本很难起作用。（实际上上面那个袜子图就是个很极端的均匀集合，现实情况远比这个复杂）

例如下图中的点，它们会在某些区域集中聚集，在其他位置又非常离散，如果使用上述的滚球法，你会遇到以下矛盾：
- 1.设置比较大的半径，否则你根本滚不到离散的点。
- 2.如果设置比较大的半径，你无法在密集点处滚出尽量多的点到凹包边中，因为直接就会被远处的角度更小的点卡主，跳过密集的点。

<p>
<image src="/convex-hull/15.png" style="width:500px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

所以，我们引入自动调节滚球半径的机制，在初始的时候我们设置尽量小的半径，以使我们尽量贴近细节的边缘前进，但是在比较稀疏的位置，我们会尝试不断拉长球的半径，直到获取到下一个合理的点，并回到原始球的大小，继续前进。

改进后的流程如下：
- 求极心 P0
- 找出屏幕最左边的一个点，这个点作为凹包边的一部分 P1
- 选中 P1，进入寻找 P2 的逻辑
    - 滚球法使用的球的半径 step（初始值100米），实际上代表的是一个扫描范围
    - 所以，下一步以 P1 为圆心，过滤出来固定半径范围内的点：nearestPoints。
    - 遍历 nearestPoints，以 P0->P1 角度为基准，逆时针旋转寻找最近的点。
        - 假设当前遍历到 P2
        - 计算 P1->P2 的角度，然后比对 P0->P1和 P1->P2 在逆时针上最接近的 P2
        - 不同：此处需加入一个过滤逻辑，我们不需要角度过小的点，因为接下去我们新增了增加滚球半径的逻辑
        - 不同：过滤逻辑，P1->P2的角度 与上一个被确定的点的角度 P1->Plast 差不能小于50，小于50的直接去掉
    - 不同：如果遍历结束最终我们没有得到 P2（经过过滤），进入递归，将 step 增加，然后继续以上一个被选中的点为 P1，寻找 P2
    - 不同：如果还没有，继续以上，将step继续增加，直到找到一个点
    - 最终递归+遍历结束后，逆时针角度最小的点就是要寻找的 P2
    - 一直执行，直到回到起点或者执行完了一周的遍历

演算视频：

<video src="/convex-hull/16.mp4" style="width:700px; border-width:3px; border-color:#666;border-style:solid;" controls autoplay muted />


## 3. 内凸包

### 3.1 内凸包定义

- 凹包没有明确的定义，内凸包也没有明确的定义
- 内凸包没有唯一值
- 内凸包的凸包不一定是绝对的凸包
- 网上关于内凸包的资料很少，讨论也比较少
- 我是这样理解内凸包的：在点的集合的一处塞入一个气球，给气球充气，在某个时刻，气球挤出去的形状就是内凸包。。
- 内凸包有一个中心参考点（气球的起点）

<p>
<image src="/convex-hull/17.png" style="width:600px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

### 3.2 内凸包场景

如下图，实现一个当鼠标在一群点中移动的时候，能够识别出内凸包，之后可以快速用比较合理的形状填充空白区域。

<p>
<image src="/convex-hull/18.png" style="width:600px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

### 3.3 内凸包计算步骤

内凸包的计算方式和凹包非常相似，我们直接采用滚球法和自动伸缩半径的方式来计算。

不过不同的是滚球不是在外围滚，而是在内部滚，要注意滚的方向。

- 设鼠标位置为 P0,P0 位于内凸包内
- 以 P0 为中心，对所有点计算其距离 P0->P，记录到每个点的信息中
- 找出 P0->P 距离最短的点作为起点 P1，这个点一定是内凸包边上的点。
- 选中 P1，进入寻找 P2 的逻辑
    - 滚球法使用的球的半径 step（初始值100米），实际上代表的是一个扫描范围
    - 所以，下一步以 P1 为圆心，过滤出来固定半径范围 step 内的点：nearestPoints。
    - 遍历 nearestPoints，以 P0->P1 角度为基准，逆时针旋转寻找最近的点。
        - 假设当前遍历到 P2
        - 计算 P1->P2 的角度，然后比对 P1->P0 和 P1->P2 在逆时针上最接近的 P2
        - 此处需加入一个过滤逻辑，我们只需要凸包，任何凹陷都要过滤掉
        - 过滤逻辑，P1->P2的角度 与上一个被确定的点的角度 P1->Plast 的角度不能大于180度，大于180度则过滤
    - 如果遍历结束最终我们没有得到 P2（经过过滤），进入递归，将 step 增加，然后继续以上一个被选中的点为 P1，寻找 P2
    - 如果还没有，继续以上，将 step 继续增加，直到找到一个点
    - 最终递归+遍历结束后，逆时针角度最小的点就是要寻找的 P2
    - 一直执行，直到回到起点或者执行完了一周的遍历

示意如下：

<p>
<image src="/convex-hull/19.png" style="width:600px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>

演算视频：

<video src="/convex-hull/20.mp4" style="width:700px; border-width:3px; border-color:#666;border-style:solid;" controls autoplay muted />

## 4. 群讨论

<p>
<image src="/convex-hull/30.png" style="width:600px;border-width:3px; border-color:#666;border-style:solid;"/>
</p>