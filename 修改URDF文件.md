### 修改URDF文件

>   对内项目一般urdf文件都是从solidworks的export_to_urdf插件导出的
>
>   为了适应moveit工程，需要固定的对raw urdf做出一些处理，从而使项目更加规范
>
>   步骤有：
>
>   1.   确保命名规范
>   2.   功能包修改为ROS2格式
>   3.   检查urdf文件
>   4.   修改关节配置（属性、限制、零点、轴的方向）
>   5.   添加word_joint
>   6.   添加tcp_link（对于夹爪）

具体的步骤有：

-   修改urdf文件名称为具体型号名称（同 robot_name ），功能包名称为粗略类型+_description 

-   这里给出例子 ec66.urdf、<robot name="ec66">、功能包名称为elite_description

-   删除base_link惯性标签，更改功能包为ROS2格式（更换CMakelists.txt和package.xml）

-   检查urdf最基础的是否导出正确，能否正确的按照轴转动，同时关节的坐标系正确，要是不正确让机械重新导出urdf

-   各关节属性先设置为revolute

    范围限制先给上-3.14到3.14

    用urdf_visualize插件导出查看是否有问题，校准好零点

    使用moveit_setup_assitant中的pose得到各关节的限制并更改属性为

-   添加word_joint

    ~~~urdf
      <joint name="world_joint" type="fixed">
        <origin xyz="0 0 0" rpy="0 0 0" />
        <parent link="world" />
        <child link="base_link" />
      </joint>
    ~~~

-   建立tcp_link（作为eef_link）作为几何中心抓取点

    ~~~urdf
     <link name="tcp_link">
        <inertial>
          <mass value="0.0001"/>
          <origin xyz="0 0 0"/>
          <inertia ixx="1e-6" ixy="0" ... />
        </inertial>
      </link>
    
      <joint name="gripper_to_tcp_joint" type="fixed">
        <parent link="finger_link"/> 
        <child  link="tcp_link"/>
        <origin xyz="0 0 0.05" rpy="0 0 0"/>
      </joint>
    
    </robot>
    ~~~

-   从sw导出的stl是高精度的，需要减少模型面数

    将stl转为dae，让urdf中vision使用dae文件

    并降低stl精度