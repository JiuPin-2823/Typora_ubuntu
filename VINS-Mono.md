### VINS-Mono

------

### 环境

- 小鱼ROS：

  ```
  wget http://fishros.com/install -O fishros && . fishros
  ```











------

### 调试

1. src/VINS-Mono/camera_model/src/calib/CameraCalibration.cc

   ```c++
   //适配opencv4,解决error: ‘CV_XXX’ was not declared in this scope
   #define CV_GRAY2RGB (cv::COLOR_GRAY2RGB)
   #define CV_AA (cv::LINE_AA)
   ```

   

2. src/VINS-Mono/camera_model/src/chessboard/Chessboard.cc

   ```c++
   //适配opencv4,解决error: ‘CV_XXX’ was not declared in this scope
   #define CV_GRAY2BGR (cv::COLOR_GRAY2BGR) 
   #define CV_BGR2GRAY (cv::COLOR_BGR2GRAY) 
   #define CV_CALIB_CB_ADAPTIVE_THRESH (cv::CALIB_CB_ADAPTIVE_THRESH ) 
   #define CV_CALIB_CB_NORMALIZE_IMAGE (cv::CALIB_CB_NORMALIZE_IMAGE ) 
   #define CV_CALIB_CB_FILTER_QUADS (cv::CALIB_CB_FILTER_QUADS )
   #define CV_CALIB_CB_FAST_CHECK (cv::CALIB_CB_FAST_CHECK )
   #define CV_ADAPTIVE_THRESH_MEAN_C (cv::ADAPTIVE_THRESH_MEAN_C )
   #define CV_THRESH_BINARY (cv::THRESH_BINARY )
   
   #define CV_SHAPE_CROSS ( cv::MORPH_CROSS )
   #define CV_SHAPE_RECT ( cv::MORPH_RECT )
   #define CV_TERMCRIT_EPS ( cv::TermCriteria::EPS )
   #define CV_TERMCRIT_ITER ( cv::TermCriteria::MAX_ITER )
   #define CV_RETR_CCOMP ( cv::RETR_CCOMP)
   #define CV_CHAIN_APPROX_SIMPLE ( cv::CHAIN_APPROX_SIMPLE)
   #define CV_THRESH_BINARY_INV ( cv::THRESH_BINARY_INV)
   ```

3. src/VINS-Mono/camera_model/src/intrinsic_calib.cc

   ```c++
   //适配opencv4,解决error: ‘CV_XXX’ was not declared in this scope
   #define CV_AA (cv::LINE_AA)
   ```

   

4. src/VINS-Mono/pose_graph/src/ThirdParty/DVision/BRIEF.cpp

   ```c++
   //适配opencv4,解决error: ‘CV_XXX’ was not declared in this scope
   #define CV_RGB2GRAY (cv::COLOR_RGB2GRAY) 
   #define CV_FONT_HERSHEY_SIMPLEX (cv::FONT_HERSHEY_SIMPLEX) 
   ```

5. src/VINS-Mono/pose_graph/src/keyframe.cpp

   ```c++
   //适配opencv4,解决error: ‘CV_XXX’ was not declared in this scope
   #define CV_FONT_HERSHEY_SIMPLEX (cv::FONT_HERSHEY_SIMPLEX) 
   ```

6. src/VINS-Mono/pose_graph/src/pose_graph.cpp

   ```c++
   //适配opencv4,解决error: ‘CV_XXX’ was not declared in this scope
   #define CV_FONT_HERSHEY_SIMPLEX (cv::FONT_HERSHEY_SIMPLEX) 
   ```

------











### 运行

1. 编译（调试报错）

2. 下载数据集[The EuRoC MAV Dataset](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets#downloads)

3. 在xxx_ws文件夹下打开三个终端，并分别输入：

   - 第一个终端

     ```shell
     source devel/setup.bash
     roslaunch vins_estimator euroc.launch 
     ```

   - 第二个终端

     ```shell
     source devel/setup.bash
     roslaunch vins_estimator vins_rviz.launch
     ```

   - 第三个终端

     ```shell
     source devel/setup.bash
     rosbag play MH_01_easy.bag //后面是数据集的路径
     ```

   - 查看节点间关系（任意终端，新开即可）

     ```shell
     rqt_graph
     ```

     

4. 命令解释[roslaunch命令](https://blog.csdn.net/qq_38364548/article/details/123526774)

   ​	
   









### 论文

#### 	预处理

1. ------

   ##### 视觉前端

   - 特征跟踪：KLT稀疏光流法

   - 特征检测：

     1. 提取特征：

        - 在两个相邻特征之间设置最小间隔以保证均匀分布
        - 每幅图像至少100-300个特征

     2. 去畸变

     3. 外点去除：RANSAC

     4. 投影：投影到一个归一化球面上

        

   - 关键帧选取：

     1. 如果当前帧和最新关键帧之间跟踪的特征点的平均视差超出某个特定阈值，则将该帧视为新的关键帧

        - 平移和旋转都会出现视差，但是只有旋转运动时，特征点无法被三角化（triangulated），因此，使用陀螺仪测量数据进行短时积分来补偿旋转

          [^triangulated]: 三角测量：在不同位置对同一个路标点进行观测，从而推断路标点的距离【14讲，7.5】

        - 这里的旋转补偿仅用于关键帧选取，不参与VINS公式中的旋转计算

        - 在这种情况下，即使陀螺仪存在较大的噪声或者偏置，也只会造成次优的关键帧选择结果，不会直接影响估计结果

          

     2. 如果跟踪的特征数量低于某一阈值，则将此帧视为新的关键帧

   

2. ------

   ##### IMU预积分

   [^参考内容]: 黑体公式编号出自《自动驾驶与机器人中的SLAM》

   

   - IMU噪声和偏置
   
   - IMU预积分
   
     ![image-20231017110555436](/home/jiupin/Typora/VINS-Mono.assets/image-20231017110555436.png)
   
     1. α、β、γ分别对应P、V、Q（位置，速度，旋转），在$$t_k $$到$$t_k+1$$内：
        - β的表示和**式4.4b**类似，只不过这里是速度的增量
        - α则是在β的基础上再做一次积分，即可表示这段时间内的位置变化量
        - γ中的$$\frac{1}{2}$$则是由于这里使用四元数表示旋转
   
     2. γ的表达式来历不是很理解
   
        
   
   - 偏置修正
   
     ![image-20231017112917684](/home/jiupin/Typora/VINS-Mono.assets/image-20231017112917684.png)
   
     - α和β的修正和**式4.32**中的p和v对应
   
     - γ的修正和**式4.32**中的R对应，只不过由于是用四元数表示的，所以利用**2.2.3**中的转换技巧，将修正量转换为四元数
   
       [^转换]: $$q = [cos(\frac{\theta}{2}),\vec{n}sin(\frac{\theta}{2})]$$，当$$\theta$$很小时，$$cos(\frac{\theta}{2})\approx1,sin(\frac{\theta}{2})\approx\frac{\theta}{2}$$，这里 的$$\theta$$就是上式中旋转修正量
   
       

​	







#### 估计器初始化

1. ------

   ##### 滑动窗口中的纯视觉sfm

   ​	由于计算复杂度受限，文章在滑动窗口中保留了几个帧。

   1. ​	在最新的帧和滑动窗口中的所有帧中检测特征，如果可以发现超过30个能够稳定跟踪的特征超过20个有效视差像素，则使用五点法恢复这两个帧之间的相对旋转和尺度平移；***否则，我们将最新的帧保存在窗口中，并等待新的帧（原文中没有 “否则...”）***

   2. 任意设置尺度，并对这两个帧中观察到的所有特征进行三角化

   3. 基于这些三角特征，使用一种PnP算法来估计窗口中所有其他帧的姿态

      [^PnP算法]: 求解3D到2D点对运动的方法，描述了当知道n个3D空间点及其投影位置时，如何估计相机的位姿【14讲，7.7，P180】

   4. 使用全局光束平差法（BA）最小化所有的特征重投影误差

      [^BA]: 对PnP问题采取非线性优化的方式，构建最小二乘问题并迭代求解【14讲，7.7，P180；14讲，9.2.2】

      

   5. 由于还没有任何世界坐标系的知识，将相机的第一个帧$$(·)^{c_0}$$设置为SfM的参考帧（reference frame）

   6. 假设相机和IMU之间有一组粗略的外参$$(p^b_c,q^b_c)$$那么可以将位姿从相机坐标系转换到IMU坐标系：

      ![image-20231017201929253](/home/jiupin/Typora/VINS-Mono.assets/image-20231017201929253.png)

      这里的s是一个用于缩放的尺度参数

      

2. ------

   ##### 视觉惯性校准

   基本思路：匹配视觉尺度结构和IMU预积分

   1. 陀螺仪偏置校准：

      考虑窗口中的连续两帧$$b_k$$和$$b_{k+1}$$：

      从视觉SfM中获得旋转$$q^{c_0}_{b_k}$$和$$q^{c_0}_{b_{k+1}}$$，并从IMU中获得相对约束$$\widetilde{\gamma}^{b_k}_{b_{k+1}}$$，对陀螺仪偏置进行预积分线性化，并最小化下面的代价函数：

      ![image-20231017194546283](/home/jiupin/Typora/VINS-Mono.assets/image-20231017194546283.png)

      [^疑问]: **这里的代价函数是怎么出来的**

      

      得到陀螺仪偏置的初始校准后即可更新IMU预积分

      **这里的代价函数是怎么出来的**

   2. 速度、重力向量和尺度初始化：

      ![image-20231017195255562](/home/jiupin/Typora/VINS-Mono.assets/image-20231017195255562.png)

      这里的重力向量使用的是第一帧中的重力向量；s将单目SfM缩放至公制单位

      

      考虑窗口中的连续两帧$$b_k$$和$$b_{k+1}$$：

      ![image-20231017195611281](/home/jiupin/Typora/VINS-Mono.assets/image-20231017195611281.png)

      观察下面的β表达式，可以理解为：先将k+1和k时刻都速度都转换到$$c_0$$时刻进行处理，处理完成之后再还原回k时刻的坐标系下；

      而阿尔法表达式中，括号内部的量本来就在$$c_0$$坐标系下，因此直接在处理完成之后将其转换到k时刻的坐标系即可；

      然后联立（6）和（9），可得到下面的测量模型：

      ![image-20231017202014187](/home/jiupin/Typora/VINS-Mono.assets/image-20231017202014187.png)

      [^疑问]: **这里不是很理解联立过程以及H的推导**

      

      由此测量模型可以抽象出以下最小二乘问题，求解即可得到窗口中每帧的速度、重力向量和尺度参数

      ![image-20231017202334315](/home/jiupin/Typora/VINS-Mono.assets/image-20231017202334315.png)

      

      

   3. 重力细化:

      1. 重力大小通常是已知的，因此只需要对重力的方向进行细化，在正切空间中只需要两个变量即可：

         $$g(\overline{\hat{g}}+\delta\vec{g})$$ ，其中，$$\delta\vec{g}=w_1\vec{b_1}+w_2\vec{b_2}$$

         $$g$$描述了重力大小，后面两个量则修正细化了重力的方向，这里的$$\vec{b_1}，\vec{b_2}$$通过下面的算法找到：

         ![image-20231019111758980](/home/jiupin/Typora/VINS-Mono.assets/image-20231019111758980.png)

         ![image-20231019111023286](/home/jiupin/Typora/VINS-Mono.assets/image-20231019111023286.png)

      2. 将细化后的重力向量加入（9）并求解，不断迭代直到 $$\hat{g}$$​ 收敛（和其他量一起最小二乘）

         

   4. 完成初始化：











#### 紧耦合单目VIO

------

1. ##### 公式

   滑动窗口中的状态向量：

   ![image-20231019112903795](/home/jiupin/Typora/VINS-Mono.assets/image-20231019112903795.png)

2. ##### IMU测量残差

3. ##### 视觉测量残差