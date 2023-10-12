### VINS-Mono：

------

#### 环境：

- 小鱼ROS：

  ```
  wget http://fishros.com/install -O fishros && . fishros
  ```



------

#### 调试：

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
