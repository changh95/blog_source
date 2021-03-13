---
title: 2021년 1월 SLAM 뉴스
date: 2021-01-28 20:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
categories: 
- [SLAM, 논문 소식]
excerpt: 2021년 1월 SLAM 논문 소식을 정리했습니다.
---

논문 이름 누르면 자세한 정보가 열립니다!

# 이번 달 내가 관심가지는 논문들

## 시스템

<details>
  <summary> Kimera: from SLAM to Spatial Perception with 3D Dynamic Scene Graphs </summary>
  
- [논문 링크](https://arxiv.org/abs/2101.06894)
- MIT의 Luca Carlone 교수님 연구
- Dynamic Scene Graph 라는 개념을 실제로 구현한 논문이다.
- 'Kimera includes ... visual-inertial SLAM, metric-semantic 3D reconstruction, object localization, human pose and shape estimation, and scene parsing'... OMG... 이전에 개발되던 Kimera의 끝판왕을 완성한 것 같다.
- 처음에 나는 Dynamic Scene Graph는 자율주행에서 만드는 HD-Map의 indoor 버전으로 보았다. 
  - HD-Map과 DSG 모두 Localization을 위한 point cloud layer를 가지고 있다. 또 semantic layer를 HD-Map은 차선과 road sign으로 구현하고, DSG는 object-level map을 구현했다는 비스무리한 공통점이 있다고 생각했는데...
  - 하지만 이번 논문으로 생각이 조금 달라졌다. 오히려 DSG가 더 큰 개념인 느낌? Hierarchical graph를 통해서 구조를 정의하고 search를 빠르게 할 뿐만이 아니라, semantic 한 정보를 저장하는 것도 훨씬 많을 수 있다는 생각이 든다. 
    - HD-Map에서 road sign 정보를 저장하는 것은 정말로 유턴, 좌회전 정보만을 저장하겠지만, DSG에서 object에 대한 정보를 적는거는 object에 대한 모든 정보를 저장하고 수많은 인터페이스를 연결할 수 있다. 
    - 예를 들어, 노트북에 대한 정보에 유저 auth 정보 등을 함께 저장한다면 정말로 '노트북켜줘' 라는 말로 버츄얼 디스플레이를 띄워서 컴퓨터에 로그인 하는 방법도 가능할거라는 생각이 든다. 
    - 물론 HD-Map에도 장소에 대한 정보 + 인터페이스를 추가할 수 있겠지만, 인도어 환경인 DSG가 조금 더 가깝게 semantic + interface 정보를 저장할 수 있지 않을까 싶다. 
- 근데 논문 왤케 길어~~~~  

</details>

<details>
  <summary> NodeSLAM: Neural Object Descriptors for Multi-View Shape Reconstruction </summary>
  
- [논문 링크](https://ieeexplore.ieee.org/abstract/document/9320317)
- ICL의 Andrew Davison 교수님의 연구
- 이미 알려질대로 알려진 NodeSLAM 연구이지만, 제대로 publication으로 나온 것 같다. Neural shape descriptor를 code (오토인코더를 통해 벡터로 압축시켜 optimise를 가능하게 만든 형태)로 만들고 RGB-D 정보와 Uncertainty를 사용해서 camera pose, object shape, object pose, map shape를 한번에 최적화 하는 논문이다.
- 이전의 연구인 MoreFusion은 Shape prior를 알고있는 상황, Fusion++은 shape prior를 전혀 모르는 상황에서 사용했었다. MoreFusion의 경우 받았던 critic은 '정확하게 shape prior를 모르는 경우에는 어쩔꺼냐?' 라는 것이였고, Fusion++에 대한 critic은 'instance segmentation으로 뽑은거는 너무 generalise된 것이 아닌가... shape 추정이 너무 안된다, 결과물이 완전 체리픽킹이다' 라는 이야기를 들었다. 
  - 이 두개의 critic을 한번에 깨주는 논문이 NodeSLAM이지 않을까 싶다. '정확한 shape prior는 없지만 대충 어떤 class가 있을 것인지 알고 있고, 해당 class에 대한 Shape은 최적화를 통해 정확히 얻어낸다. 그렇기 때문에 수많은 shape들에도 사용할 수 있고, 정확도도 높다!' 라는것을 보여주는데... 
  - 역시 감탄만 나오는 Davison 교수님 (물개박수)

</details>

<details>
  <summary> LOCUS: A Multi-Sensor Lidar-Centric Solution for High-Precision Odometry and 3D Mapping in Real-Time </summary>

- [논문 링크](https://ieeexplore.ieee.org/abstract/document/9293359/)
- MIT의 Luca Carlone 교수님 랩실 논문은 못참지
- 아카이브에 안뜨려나...

</details>

<details>
  <summary> Local to Global: Efficient Visual Localization for a Monocular Camera </summary>

- [논문 링크](https://openaccess.thecvf.com/content/WACV2021/papers/Lee_Local_to_Global_Efficient_Visual_Localization_for_a_Monocular_Camera_WACV_2021_paper.pdf)
- 네이버 랩스 + ICCV 2019년도 Point & Line SLAM의 이상준님의 논문
- Visual localization 연구는 대부분 Hierarchical localization 방식을 따라 많이들 NetVLAD + SuperPoint 방식을 사용한다. 
  - 이 방식은 잘 되지만 (?), SuperPoint가 아직 모바일 디바이스에서 실시간으로 슈슉 돌아가기 어렵다는 단점이 있다. 
  - 아무래도 연구자들은 '하드웨어 개발이 해결해주겠지' 라고 생각을 하는 것 같다. 
- 이번 연구를 통해서 실시간 VO에 많이 사용되는 ORB를 SuperPoint을 조합하여 실시간성 + 정확도를 둘 다 잡아보는 SuperORB + SuperKeyframe 이라는 연구를 보여준다. 
- 논문을 보면 저자 분이 VO 시스템에 대해 깊은 이해를 가지고 있다는 것을 볼 수 있다 (ㄷㄷ) 소스 코드 한번만 볼 수 있다면 소원이 없을듯

</details>

<details>
  <summary> ALVIO: Adaptive Line and Point Feature-based Visual Inertial Odometry for Robust Localization in Indoor Environments </summary>

- [논문 링크](https://arxiv.org/pdf/2012.15008.pdf)
- 카이스트에서 나온 VIO

</details>

<details>
  <summary> VINS-PL-Vehicle: Points and Lines-based Monocular VINS Combined with Vehicle Kinematics for Indoor Garage </summary>

- [논문 링크](https://ieeexplore.ieee.org/abstract/document/9304639)
- VIO에 visual keypoint와 line을 함께 쓰는 연구

</details>

<details>
  <summary> High Precision Vehicle Localization based on Tightly-coupled Visual Odometry and Vector HD Map </summary>

- [논문 링크](https://ieeexplore.ieee.org/abstract/document/9304659)
- VIO에 Vector HD-Map을 하나의 센서처럼 estimation에 포함시킨 연구

</details>

<details>
  <summary> A Versatile Keyframe-Based Structureless Filter for Visual Inertial Odometry </summary>

- [논문 링크](https://arxiv.org/pdf/2012.15170.pdf)

</details>

<details>
  <summary> Object-Level Semantic Map Construction for Dynamic Scenes </summary>

- [논문 링크](https://www.mdpi.com/2076-3417/11/2/645/pdf)
- 3D 모델 정보를 사용해서 detection을 하고 object tracking을 하고 SLAM을 하는 연구.
  - MoreFusion과 비슷한 연구?

</details>

<details>
  <summary> Using Detection, Tracking and Prediction in Visual SLAM to Achieve Real-time Semantic Mapping of Dynamic Scenarios </summary>

- [논문 링크](https://ieeexplore.ieee.org/abstract/document/9304693)
- ORB-SLAM2에 Object detection을 추가하고 estimation을 돌린 연구.

</details>

<details>
  <summary> DeepSurfels: Learning Online Appearance Fusion </summary>

- [논문 링크](https://arxiv.org/pdf/2012.14240.pdf)
- ETH Zurich와 Microsoft Mixed Reality Labs의 Marc Pollefeys 교수님 
- 조금 더 읽어봐야할듯...

</details>


## 서베이

<details>
  <summary> Survey and Evaluation of RGB-D SLAM </summary>

- [논문 링크]((https://ieeexplore.ieee.org/abstract/document/9330596/))
- Survey 논문은 언제나 환영!
- 최근 딥러닝 기반 Single image depth estimation 기능을 통해 monocular camera로도 depth map을 얻어내어 visual odometry에 포함시키는 연구가 많이 진행되고 있다. 
  - 물론 딥러닝 기반 depth estimation은 센서로 얻어낸 정보보다는 많이 부정확하기에 기존의 RGB-D SLAM 파이프라인을 쓰는 것 처럼 사용하기는 어렵다. 
  - Depth 값을 error cost로 사용해 정확한 모델을 얻어내기보다는 search space를 줄여주는 하나의 가이드 정도로 사용해준다면, 아니면 depth 값을 사용하여 surfel 값을 얻어낸다던지 free space를 찾는다던지로 monocular에서는 불가능했던 새로운 기능을 추가할 수 있지 않을까 생각을 하게 된다.
- 이런 방식들을 실제로 구현해내려고 한다면 기존의 RGB-D SLAM에 대한 이해가 필요하다고 보는데, 이 survey 논문이 굉장히 깊게 설명해주는 것 같다. 아이러브 서베이! ::inlove::

</details>

<details>
  <summary> A Survey on 3D LiDAR Localization for Autonomous Vehicles </summary>

- [논문 링크](https://ieeexplore.ieee.org/abstract/document/9304812)
- LiDAR 데이터 localization 방식을 정리해준 survey 논문이라고??
- Survey 논문은 사랑입니다

</details>

<details>
  <summary> Single-View 3D reconstruction: A Survey of deep learning methods </summary>

- [논문 링크](https://www.sciencedirect.com/science/article/abs/pii/S0097849320301849)
- Single-view 3D reconstruction은 사실 잘 모르는 분야이지만, 어떻게보면 SLAM의 목적이 어느정도 이 분야에 들어가있다고 볼 수 있다.
- Multi-view geometry로 3D reconstruction을 하는것이 Structure-from-Motion과 SLAM인데...
  - 그걸 Single-view로 하는 연구는 딥러닝만 가능하다고 보고 있다.
  - 물론 아직 multi-view가 정확도가 더 뛰어난 편이 많기 때문에 (그리고 그 정확도는 localization에 중요하기 때문에) 아직 이 분야를 깊게 보지는 않았다.
- 근데 보기 쉽게 survey로 만들어주다니!
  - Survey 논문은 사랑입니다(3)
- SLAM쪽에서는 대부분 point cloud나 voxel, octree 데이터를 다루는데, 이 논문에서는 추가로 mesh, implicit surfaces, primitive 정보까지 다룬다.

</details>

<details>
  <summary> Deep learning, Inertial Measurements Units, and Odometry: Some Modern Prototyping Techniques for Navigation Based on Multi-Sensor Fusion </summary>

- [논문 링크](https://www.researchgate.net/profile/Martin_Brossard/publication/345988825_Deep_learning_Inertial_Measurements_Units_and_Odometry_Some_Modern_Prototyping_Techniques_for_Navigation_Based_on_Multi-Sensor_Fusion/links/5fc10070458515b79778a395/Deep-learning-Inertial-Measurements-Units-and-Odometry-Some-Modern-Prototyping-Techniques-for-Navigation-Based-on-Multi-Sensor-Fusion.pdf)
- 이거 왜 EKF 설명 잘되어있지...?
- 근데 박사 논문이라서 211페이지 허우...

</details>


## Relative / Absolute pose estimation

<details>
  <summary> A Complete, Accurate and Efficient Solution for the Perspective-N-Line Problem </summary>

- [논문 링크](https://ieeexplore.ieee.org/abstract/document/9310278)
- CMU의 Michael Kaess 교수님 논문 연구
- Perspective-N-Line이라면, line 들 정보를 통해서 absolute camera pose를 찾을 수 있다는걸까? 
  - Line 기반 트랙킹 기술을 한동안 보았는데, 이 기술과 함께 써서 visual keypoint가 부족한 실내 corridor 환경 등등에서 SLAM을 할 수 있지 않을까 라는 상상을 해본다. 
  - Point & line SLAM 꼭 한번 제대로 만들어보고싶다.

</details>

<details>
  <summary> Fast and Robust Certifiable Estimation of the Relative Pose Between Two Calibrated Cameras </summary>

- [논문 링크](https://arxiv.org/pdf/2101.08524.pdf)
- Global optimum이 되는 relative pose estimation을 구하는 방법
  - RANSAC보다 (당연히) 정확함

</details>

<details>
  <summary> Hybrid Rotation Averaging: A Fast and Robust Rotation Averaging Approach </summary>

- [논문 링크](https://arxiv.org/pdf/2101.09116.pdf)
- Laurent Kneip 교수님 연구실
- Rotation averaging 에서 Local method와 Global method가 있다고 한다.
  - 보통 Local method를 사용하고, global method는 느려서 잘 안쓴다고 한다.
- 이 논문에서는 Global의 방식과 Local 방식의 장점을 혼합한 hybrid 방식을 제안한다고 한다.

</details>
  
<details>
  <summary> Provably Approximated ICP </summary>

- [논문 링크](https://arxiv.org/pdf/2101.03588.pdf)
- Point cloud간의 transformation을 구할 때 global optimum을 찾는 방식을 증명하고 구하는 방식을 제안한 연구.

</details>

<details>
  <summary> On the Tightness of Semidefinite Relaxations for Rotation Estimation </summary>

- [논문 링크](https://arxiv.org/pdf/2101.02099.pdf)
- Viktor Larsson 교수님 연구
- Semi-definite relaxation...?

</details>

<details>
  <summary> A Linear Approach to Absolute Pose Estimation for Light Fields </summary>

- [논문 링크](http://users.ics.forth.gr/~lourakis/publ/2020_3dv.pdf)
- Lightfield camera에 특화된 absolute camera pose estimation 기법을 제안.

</details>

<details>
  <summary> Factor Graphs: Exploiting Structure in Robotics </summary>

- [논문 링크](https://www.annualreviews.org/doi/abs/10.1146/annurev-control-061520-010504)
- 논문 아직 안나옴 (...)

</details>

<details>
  <summary> Comparison of camera-based and 3D LiDAR-based place recognition across weather conditions </summary>

- [논문 링크](https://ieeexplore.ieee.org/abstract/document/9305429)
- 카메라 vs LiDAR vs 카메라+LiDAR 퓨전 방식의 place recognition을 비교하는 연구.

</details>

## 딥러닝

<details>
  <summary> QuadroNet: Multi-Task Learning for Real-Time Semantic Depth Aware Instance Segmentation </summary>

- [논문 링크](https://openaccess.thecvf.com/content/WACV2021/papers/Goel_QuadroNet_Multi-Task_Learning_for_Real-Time_Semantic_Depth_Aware_Instance_Segmentation_WACV_2021_paper.pdf)
- Zoox의 연구

</details>

<details>
  <summary> Boosting Monocular Depth with Panoptic Segmentation Maps </summary>

- [논문 링크](https://openaccess.thecvf.com/content/WACV2021/papers/Saeedan_Boosting_Monocular_Depth_With_Panoptic_Segmentation_Maps_WACV_2021_paper.pdf)

</details>
  
<details>
  <summary> Real-Time Uncertainty Estimation in Computer Vision via Uncertainty-Aware Distribution Distillation </summary>

- [논문 링크](https://openaccess.thecvf.com/content/WACV2021/papers/Shen_Real-Time_Uncertainty_Estimation_in_Computer_Vision_via_Uncertainty-Aware_Distribution_Distillation_WACV_2021_paper.pdf)

</details>

<details>
  <summary> Self-Supervised Pretraining of 3D Features on any Point-Cloud </summary>

- [논문 링크](https://arxiv.org/pdf/2101.02691.pdf)
- 3D 포인트 클라우드를 사용하기 위해 pretraining하는 기법에 대해 소개함.

</details>


# 시간이 되면 읽어볼 논문들

- [VIO-Aided Structure from Motion Under Challenging Environments](https://arxiv.org/pdf/2101.09657.pdf)
- [Monocular Visual Inertial Direct SLAM with Robust Scale Estimation for Ground Robots/Vehicles](https://www.mdpi.com/2218-6581/10/1/23/htm)
- [On Camera Pose Estimation for 3D Scene Reconstruction](https://dl.acm.org/doi/abs/10.1145/3430984.3431038)
- [CoMoDA: Continuous Monocular Depth Adaptation Using Past Experiences](https://openaccess.thecvf.com/content/WACV2021/papers/Kuznietsov_CoMoDA_Continuous_Monocular_Depth_Adaptation_Using_Past_Experiences_WACV_2021_paper.pdf)
- [Self-supervised Visual-LiDAR Odometry with Flip Consistency](https://openaccess.thecvf.com/content/WACV2021/papers/Li_Self-Supervised_Visual-LiDAR_Odometry_With_Flip_Consistency_WACV_2021_paper.pdf)
- [Neural vision-based semantic 3D world modeling](https://openaccess.thecvf.com/content/WACV2021W/AVV/papers/Papadopoulos_Neural_Vision-Based_Semantic_3D_World_Modeling_WACVW_2021_paper.pdf)

- [A Semi-Direct Monocular Visual SLAM Algorithm in Complex Environments](https://link.springer.com/article/10.1007/s10846-020-01297-8)
- [An Efficient Iterated EKF-Based Direct Visual-Inertial Odometry for MAVs Using a Single Plane Primitive](https://ieeexplore.ieee.org/abstract/document/9309401)
- [Global visual and Semantic Observations for Outdoor Robot Localization](https://ieeexplore.ieee.org/abstract/document/9296251)
- [Research on the Availability of VINS-Mono and ORB-SLAM3 Algorithms for Aviation](https://www.researchgate.net/profile/Metin_Turan3/publication/347937332_Research_on_the_Availability_of_VINS-Mono_and_ORB-SLAM3_Algorithms_for_Aviation/links/5fe8d59e299bf140885031f7/Research-on-the-Availability-of-VINS-Mono-and-ORB-SLAM3-Algorithms-for-Aviation.pdf)
- [UPSLAM: Union of Panoramas SLAM](https://arxiv.org/pdf/2101.00585.pdf)
- [Visual-IMU State Estimation with GPS and OpenStreetMap for Vehicles on a Smartphone](https://ieeexplore.ieee.org/abstract/document/9305386)
- [CNN-based Visual Ego-Motion Estimation for Fast MAV Maneuvers](https://arxiv.org/pdf/2101.01841.pdf)
- [Homography-based camera pose estimation with known gravity direction for UAV navigation](http://scis.scichina.com/en/2021/112204.pdf)
- [On-the-fly Extrinsic Calibration of Non-Overlapping in-Vehicle Cameras based on Visual SLAM under 90-degree Backing-up Parking](https://ieeexplore.ieee.org/abstract/document/9304530)
- [Single-Shot 3D Detection of Vehicles from Monocular RGB Images via Geometrically Constrained Keypoints in Real-Time](https://ieeexplore.ieee.org/abstract/document/9304847)
- [TIMA SLAM: Tracking Independently and Mapping Altogether for an Uncalibrated Multi-Camera System](https://www.mdpi.com/1424-8220/21/2/409/pdf)
- [Deep regression for LiDAR-based localization in dense urban areas](https://www.sciencedirect.com/science/article/abs/pii/S0924271620303518)
- [Stereo camera visual SLAM with hierarchical masking and motion-state classification at outdoor construction sites containing large dynamic objects](https://www.tandfonline.com/doi/abs/10.1080/01691864.2020.1869586)
- [A general framework for modeling and dynamic simulation of multibody systems using factor graphs](https://arxiv.org/pdf/2101.02874.pdf)
- [A SLAM System Based on RGBD Image and Point-Line Feature](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9316224)
- [PLUME: Efficient 3D Object Detection from Stereo Images](https://arxiv.org/pdf/2101.06594.pdf)
- [Asynchronous Multi-View SLAM](https://arxiv.org/pdf/2101.06562.pdf)
- [Incremental Rotation Averaging](https://link.springer.com/article/10.1007/s11263-020-01427-7)
- [PL-GM:RGB-D SLAM With a Novel 2D and 3D Geometric Constraint Model of Point and Line Features](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9316710)
- [DP-SLAM: A visual SLAM with moving probability towards dynamic environments](https://www.researchgate.net/profile/Zonghai_Chen/publication/348256049_DP-SLAM_A_visual_SLAM_with_moving_probability_towards_dynamic_environments/links/5fff9c3845851553a0418106/DP-SLAM-A-visual-SLAM-with-moving-probability-towards-dynamic-environments.pdf)
- [Visual SLAM for robot navigation in healthcare facility](https://www.sciencedirect.com/science/article/pii/S0031320321000091)
- [Panoramic Visual SLAM Technology for Spherical Images](https://www.mdpi.com/1424-8220/21/3/705/pdf)
- [LSTM and Filter Based Comparison Analysis for Indoor Global Localization in UAVs](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9316698)
