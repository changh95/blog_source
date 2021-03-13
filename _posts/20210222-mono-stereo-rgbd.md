---
title: Monocular, Stereo, RGB-D SLAM의 비교
date: 2021-02-22 23:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Monocular SLAM
- Stereo SLAM
- RGB-D SLAM
categories: 
- [SLAM, 50가지 CV/SLAM 기술면접 질문들]
excerpt: Monocular vs Stereo vs RGB-D SLAM 비교
---

> 모바일로 글을 보신다면 '데스크탑 버전으로 보기'를 누르시면 좀 더 보기 쉽습니다

# 예상 면접 질문: 
1. Monocular, Stereo, RGB-D SLAM의 차이점이 무엇인가요? (또는 각각의 특징, 장단점을 알려주세요)
2. Stereo나 RGB-D가 Monocular 방식에 대해 가지는 장점이 무엇일까요?

# 답

- **Monocular SLAM**
  - 카메라 1대를 사용하여 SLAM을 하는 것
    - 하드웨어가 상대적으로 stereo camera나 RGB-D camera보다 저렴함 (+ 전력 소비량도 작음).
  - Monocular camera 이미지 1장으로는 3D 공간을 up-to-scale로만 추정할 수 있음.
    - Image projection, 또는 N-view geometry로도 up-to-scale로만 추정 가능함.
      - 정확한 scale 값을 모르기 때문에 **Scale ambiguity**가 있다고 할 수 있음.
        - 이러한 Scale ambiguity는 이미 알고있는 물체의 크기를 (i.e. Fiducial 마커) 대조하거나, 아니면 IMU 센서 등으로 복원할 수 있음.
    - Stereo camera나 RGB-D camera에서는 Depth 정보를 취득할 수 있어 실제 metric scale을 얻어내는 것과는 다름.
      - 최근 딥러닝 기반 monocular depth estimation, 또는 single image monocular depth estimation 기술을 사용해 depth 정보를 추출하기도 한다. 
  - 360 카메라나 어안렌즈와 같은 특수한 렌즈 구성을 사용하지 않는 이상 field of view가 작다.
  - Monocular SLAM의 연구 방향은 대체적으로 '**하나의 이미지만으로 어느정도까지 정확하게 SLAM을 할 수 있는가**'에 초점을 둠.

<br>

- **Stereo SLAM**
  - **Baseline** 거리만큼 떨어진 카메라 2대를 사용하여 SLAM을 하는 것
    - 이 때, 카메라는 동시에 이미지를 취득해야함. (i.e. **synchronised cameras**)
    - Stereo를 계속 추가해서 붙여나가 multi-camera system도 만들 수 있음.
  - Stereo 시스템은 한 프레임에 취득한 두 이미지간의 **disparity** 정보를 이용해서 픽셀마다 depth를 얻어낼 수 있음.
    - 이 Depth를 unproject함으로써 한 프레임의 데이터만으로도 실제 metric scale의 3D structure를 얻어낼 수 있음.
      - Baseline 거리가 큰 만큼 더 먼 거리의 3D structure를 정확하게 잴 수 있음.
      - 프레임간 모션 정보를 Monocular 방식이 사용하는 방법들을 사용해서 얻을 수도 있고, **ICP**를 사용해서 얻을 수도 있음.
    - 픽셀마다 Depth를 계산하는데에 **꽤 많은 계산량**이 필요하며, 종종 실시간으로 계산하기 위해 GPU나 FGPA 등 가속장비를 사용함.
  - 카메라 간의 baseline 정보와 각각의 카메라의 렌즈 특성에 대한 **calibration**하는 작업이 대체적으로 굉장히 귀찮음.
  - Stereo SLAM이나 Multi-camera SLAM의 연구 방향은 대체적으로 '**가장 정확한 SLAM을 만드는 것**'에 초점을 둠.

<br>

- **RGB-D SLAM**
  - **Structured light** (i.e. 구조광)이나 **Time-of-Flight** (ToF) 센서를 이용한 카메라를 사용함.
    - Strutured light는 임의의 패턴을 방출하여, 이미지 센서로 해당 패턴을 읽어서 Depth를 추출함.
    - ToF 센서는 빛을 방출하고 이미지 센서로 돌아오는 시간을 측정하여 거리를 추출함.
  - 픽셀간 depth 정보를 얻는데에 Stereo 만큼 계산량이 높지 않음.
    - RGB-D에서는 센서의 기능을 계산량을 엄청나게 줄여서 실시간으로 depth 데이터를 쉽게 취득 가능함.
  - 취득한 Depth 데이터를 unproject함으로써 한 프레임의 데이터만으로도 실제 metric scale의 3D structure를 얻어낼 수 있음.
  - 단점도 꽤 많음.
    - 길어도 ~20m 정도밖에 Depth 정보를 얻지 못함. 이 데이터도 노이즈가 상당함
    - Field of view가 작음
    - 실외에서 사용하기 어려움 (적외선 파장이 햇빛에 포함된 적외선 파장과 겹침)
  - RGB-D SLAM의 연구 방향은 대체적으로 '**Dense SLAM/Mapping**' 또는 '**Depth 정보를 사용해서 어떤 새로운 기능을 만들수있을까**'에 초점을 둠.

