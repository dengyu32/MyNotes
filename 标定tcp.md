### 标定tcp

tcp (tool center point) 即工具中心点, 作为末端, 用于实际末端姿态规划

若直接使用法兰坐标系, 规划姿态时会发生位置甚至姿态错误, 需要在urdf中标定tcp

---

#### 三类坐标系

soildworks导出urdf文件时, 只需要设置每一个joint的坐标系,

但是实际上urdf确定了三类坐标系,分别是 joint \ link \ mesh frame

三者的关系为  parent_link frame -- joint origin --> joint frame -- joint运动 --> child_link
  frame -- visual/collision origin --> mesh frame

这里以joint6 link6法兰坐标系为例

**joint6 坐标系**

![image-20260417194450562](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221553620.png)

urdf 关节属性定义 link5 -> link6 这段运动链怎么接上去, 调整零点时调整origing rpy的其中一个即可

**link6 urdf 坐标系**

![image-20260417193933618](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221556794.png)

这里urdf link坐标系是运动学真正参与 FK / IK 规划的坐标系, 受 joint 移动影响

红圈框住的是mesh 挂载的 origin, 与 link frame 无关

**link6 STL mesh坐标系**

![image-20260417193452544](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221607482.png)

STL作为模型文件自己的局部坐标系, 只通过 visual/collision origin 挂载到link6, 不参与实际规划, 不用管

在sw urdf导出时导出器默认将mesh坐标系与link坐标系重合,即所有mesh坐标系均通过0 0 0 0 0 0挂载到 link 的visual 和collision

也就是说,未作旋转时这三个坐标系是相同的

![image-20260417204206598](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221616880.png)

![image-20260417204554932](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221624070.png)

所以对于添加一个tcp_link,只需通过标定tcp_joint的origin即可

---

#### 标定 tcp 坐标系

首先确定link和物体坐标系的姿态

以link6为例

这是视觉传来的物体坐标系

![image-20260417205836327](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221629742.png)

要使夹爪合理的抓取能量单元需要做一个 x -> z , y -> x , z -> y 的姿态变换,和一个z轴上的位移

算出旋转矩阵 [ 0 1 0 ] 并测量出z轴offest (11cm)

​			[ 0 0 1 ]

​			[ 1 0 0 ]

---

#### 写入urdf中

![image-20260417212003638](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221638551.png)

如图, tcp_link 添加一个仅视觉的酒红色的小球

foxglove可视化后

![](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221641763.png)

---

#### 应用到规划中

对于moveit:

eelink设为tcp_link

engineer_arm规划组中添加tcp_link

end_effector改为tcp_link

对于idl:

更改KDL Chain

![image-20260417220415158](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260417221645731.png)