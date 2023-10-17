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

1. ##### 视觉前端

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

          [^triangulated]: 三角测量是一种用于从**多个视角**重建三维场景的技术

        - 这里的旋转补偿仅用于关键帧选取，不参与VINS公式中的旋转计算

        - 在这种情况下，即使陀螺仪存在较大的噪声或者偏置，也只会造成次优的关键帧选择结果，不会直接影响估计结果

          

     2. 如果跟踪的特征数量低于某一阈值，则将此帧视为新的关键帧

2. ##### IMU预积分

   [^参考公式]: 《自动驾驶与机器人中的SLAM》

   - IMU噪声和偏置

   - IMU预积分

     ![image-20231017110555436](/home/jiupin/Typora/VINS-Mono.assets/image-20231017110555436.png)
   
     1. α、β、γ分别对应P、V、Q（位置，速度，旋转），在$$t_k $$到$$t_k+1$$内：
        - β的表示和式4.4b类似，只不过这里是速度的增量
        - α则是在β的基础上再做一次积分，即可表示这段时间内的位置变化量
        - γ中的$$\frac{1}{2}$$则是由于这里使用四元数表示旋转

     2. γ的表达式来历不是很理解
   
        
   
   - 偏置修正
   
     ![image-20231017112917684](/home/jiupin/Typora/VINS-Mono.assets/image-20231017112917684.png)
   
     - α和β的修正和式4.32中的p和v对应
   
     - γ的修正和式4.32中的R对应，只不过由于是用四元数表示的，所以利用2.2.3中的技巧，将修正量转换为四元数
   
       [^转换]: $$q = [cos(\frac{\theta}{2}),\vec{n}sin(\frac{\theta}{2})]$$，当$$\theta$$很小时，$$cos(\frac{\theta}{2})\approx1,sin(\frac{\theta}{2})\approx\frac{\theta}{2}$$
   
       

估计其

​	