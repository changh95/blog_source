---
title: 50가지 CV/SLAM 기술면접 질문 리스트
date: 2021-02-21 15:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Interview
- Questions
categories: 
- [SLAM, 50가지 CV/SLAM 기술면접 질문들]
excerpt: 나름대로 정리해본 50가지 CV/SLAM 기술면접 질문들입니다.
---

1. [Image projection 과정에 대해서 설명해주세요. Intrisic matrix와 Extrinsic matrix도 설명해주세요.](https://changh95.github.io/20210222-image-projection/)
2. [Monocular / Stereo / RGB-D SLAM의 특징을 설명해주세요. 각각 방식의 장점과 단점은 무엇인가요?](https://changh95.github.io/20210222-mono-stereo-rgbd/)
3. [SLAM에서 쓸 수 있는 센서에는 어떤게 있을까요? 알고있는 센서들 각각의 작동방법과 특징과 장단점을 이야기하세요. 이들을 함께 쓰려면 어떻게 해야할까요?](https://changh95.github.io/20210228-AD-sensors/)
4. RANSAC에 대해 설명해주세요. RANSAC의 장점과 단점도 설명해주세요. 다른 Outlier removal 기법 중 아는 방식이 있다면 설명해주세요
5. Loop closure가 무엇인가요? 어떻게 하는건가요?
6. Bundle adjustment에 대해서 설명해주세요
7. Gradient descent 방식, Newton-Raphson 방식, Gauss-Newton 방식, Levenberg-Marquardt 방식 최적화 기법은 어떻게 다른가요?
8. Kalman filter와 Particle filter에 대해서 설명해주세요. Kalman filter와 Extended Kalman filter의 차이도 함께 설명해주세요.
9. Visual SLAM과 Visual-Inertial Odometry는 무슨 차이가 있나요?
10. Tightly-coupled 방식과 Loosely-coupled 방식의 차이를 설명해주세요.
11. Fundamental matrix 와 Essential matrix가 무엇인가요? Epipolar constraint는 무엇인가요?
12. 실내 SLAM과 실외 SLAM은 어떤점들이 다를까요?
13. Rotation matrix, Quaternion, Axis-angle, Euler angle의 차이점을 설명해주세요
14. $SO(3)$와 $SE(3)$는 어떻게 다르나요?
15. Lie Group과 Lie Algebra는 왜 쓰이나요?
16. PTAM, ORB-SLAM, SVO의 차이점을 이야기해주세요.
17. Fundamental matrix와 Essential matrix의 degree of freedom은 각각 몇개인가요? 5/7/8 point 알고리즘을 설명해주세요.
18. 카메라와 IMU 캘리브레이션은 어떻게 하나요?
19. Keypoint detector와 Keypoint descriptor의 다른 점을 설명하세요. Keypoint tracking은 어떻게 수행하나요?
20. Feature의 invariance는 어떤 것을 의미하나요?
21. Homography matrix가 무엇인가요?
22. Perspective-n-Point 문제가 무엇인가요?
23. Inverse depth parameterization이 무엇인가요?
24. Feature-based method와 Direct method의 다른 점을 설명하고, 두 방식의 장단점을 설명하세요. Direct method와 Optical flow가 어떻게 다른지도 설명하세요.
25. Marginalization이 무엇인가요?
26. Floating-point descriptor와 Binary descriptor는 무슨 차이가 있나요?
27. IMU Pre-integration이 무엇인가요?
28. SLAM을 하는 도중 움직이는 물체가 있을 때 어떻게 해야할까요?
29. Graph optimization이 무엇이고, 왜 쓰나요?
30. Information matrix란 무엇인가요?
31. Sparse mapping과 Dense mapping의 차이와 장단점을 설명하세요.
32. Extended Kalman filter와 Bundle adjustment의 차이를 설명하세요.
33. Line / Edge 추출 방식들을 설명하고 각각의 장단점을 설명하세요.
34. Structure-from-Motion과 Visual odometry는 어떻게 다른가요?
35. SLAM에서 Keyframe이 무엇일까요?
36. Scale drift는 왜 나타나는걸까요?
37. 여러개의 map을 하나로 합치려면 어떻게 해야할까요?
38. Relocalization이란 무엇인가요?
39. BoW와 VLAD에 대해서 설명해주세요.
40. ICP (Iterative closest point)의 작동 방법을 설명해주세요.
41. Patch similarity를 구하는 방법 중 SSD, SAD, NCC를 비교해주세요.
42. Feature descriptor distance는 어떤 방식으로 구해지나요?
43. 이미지에서 노이즈를 줄이는 방식에 대해 소개해주세요.
44. 반복되는 패턴을 바라보고 있는 2장의 이미지가 있습니다. 이 때 두 이미지 사이의 정확한 변환관계를 어떻게 구하나요?
45. FLANN, LSH, Multi-probe LSH, kd-tree에 대해서 설명해주세요.
46. 정확한 Feature matching을 만드는 전략에는 어떤 방법들이 있을까요?
47. Image pyramid에 대해서 설명해주세요.
48. MLE와 MAP는 어떤게 다른가요?
49. DLT (Direct Linear Transform)에 대해서 설명해주세요.
50. Visual-LiDAR fusion 방식에 대해서 아는대로 얘기해주세요.


하나씩 풀어서 적어보도록 하자 :smile: