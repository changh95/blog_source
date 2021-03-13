---
title: Graphcore IPU로 Bundle adjustment / SLAM 하기!
date: 2021-01-28 22:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Graphcore
- IPU
- Bundle adjustment
categories: 
- [SLAM]
excerpt: 메가존 클라우드에서 지원을 받아 Graphcore IPU로 Bundle adjustment / SLAM을 해봤습니다.
---

# IPU로 BA / SLAM?

[Graphcore](https://www.graphcore.ai/)는 영국의 반도체 회사로, AI 계산에 특화된 프로세서인 IPU를 개발한다. 현재 단계의 SLAM은 아직 완전 AI라고 할 수 없지만, 영국 Imperial College London의 Andrew Davison 교수님은 IPU와 같은 구조의 프로세서가 Spatial AI를 만들 수 있을 것이라고 굳게 믿고 계신다. IPU에 대한 설명은 이 [링크](https://changh95.github.io/20201226-CVPR-2020-SLAM-workshop-Davison/)를 참고하자.

이에 대해 [FutureMapping 2: Gaussian Belief Propagation for Spatial AI](https://arxiv.org/abs/1910.14139)라는 논문을 통해 Bundle adjustment를 Gaussian belief propagation 형태로 풀은 선행 연구가 있었다. GBP 방식은 효과적이었으나 CPU에서는 효율적이지 않아 ceres-solver, g2o, GTSAM 등에서는 현재 지원하고 있지 않은 기능이다. 하지만 IPU의 특별한 구조 덕분에 GBP 방식을 실제로 사용할 수 있을 것 같은 가능성이 보였다.

이 논문이 발표되고 1년의 시간이 지나 개선된 IPU 제품이 출시되었으며, 2020년 CVPR에서 해당 장비를 이용하여 CPU에서의 ceres-solver 라이브러리의 성능을 뛰어넘은 [Bundle adjustment on a graph processor](https://arxiv.org/abs/2003.03134) 논문이 발표되었다. 아래는 관련 영상이다.

{% youtube TqeN8aQNgd0 %}

---

# 코드

이 논문에 꽤나 감명받아있던 때에, 국내 클라우드 업체인 [메가존 클라우드](https://www.megazone.com/)에서 Graphcore IPU 서버를 수입하고 테스터들을 모집하고 있었다. 아무래도 IPU 칩은 주로 클라우드 기반 딥러닝 학습에 초점을 두고 있다보니 모집인원은 딥러닝에 집중되어있었지만, 감사하게도 지인분께서 IPU를 사용해볼 수 있게 메가존 클라우드에 나를 소개해주셨다. 나를 포함한 3명이 첫 라운드로 하나의 서버를 할당받았고, 메가존 클라우드 측에서도 TensorFlow 환경과 PyTorch 환경을 미리 설정해주시고 Q&A 채널을 만들어주시는 등 지원을 아낌없이 해주셨다. 물론 나는 딥러닝을 할 것이 아니기 때문에, BA / SLAM 코드를 따로 끌어왔어야했다.

다행히도 Bundle adjustment on a graph processor 코드는 저자의 Github에 올라와있었다. 지원받은 기기는 M2000 IPU 속 일정 부분이였는데 (POD64와 성능이 비슷한? 또는 동일한? 것으로 알고 있다), 논문에서 구현된 내용을 돌리기에는 충분했을 것이다. ssh로 서버에 접속 후 git clone으로 끌어와서 돌려봤는데, 런타임 에러가 있었다. IPU를 다루게 해주는 프레임워크인 Poplar SDK 코드가 업데이트 되면서 바뀐건가? 했는데 그건 아닌 것 같았다. 어쩐지 조금 불신이 생겼지만, 해당 내용을 수정하고 저자의 repo에 [PR](https://github.com/joeaortiz/gbp-poplar/pull/1)을 넣었다.

{% asset_img "ipu-running.png" "BA" %}

그리고 코드를 실행해보았다. 우선 첫번째로 느낀 점은 **데이터 로딩이 무지하게 오래 걸린다는 점**이다. 거의 **20~30초**는 데이터로딩에 걸린 것 같다 (즉, 이 상태로 20~30초를 기다린다). 이게 이렇게 되는게 맞는건가...? 아직도 솔직히 의문이긴 하다.

{% asset_img "gbp-poplar.gif" "BA-2" %}

그리고는 위와 같이 코드가 돌아갔다. 이렇게 돌아가면서 느낀 점은 **여전히 너무 느리다는 것**이다. '설마 iteration 도는거를 보기 쉽게 하려고 std::chrono 같은걸 끼얹었나?' 라는 생각도 들어서 코드를 뜯어봤지만 그런건 없었다. 분명 논문에서는 250ms에 BA가 끝났다는데, 내가 돌린 코드에서는 20초가 걸렸다 ㅋㅋㅋ 이런 결과를 보면 IPU에 대한 신뢰도가 사라질 것이다. 왜냐하면 CPU에서 ceres-solver가 도는 모습을 log로 찍으면 눈에 보이지도 않기 때문이다.

---

# 저자와의 컨택

저자와 여러 이메일을 주고 받았다.

나는 저자에게 '내가 돌린 코드에서는 20초가 나오는데, 논문에서 나오는 40ms는 어떻게 측정한 것인지?' 라는 질문을 던졌다. 이에 대해 저자는 **reprojection error가 1.5 pixel보다 작은 수치로 수렴하는 시간이 40 ms**라고 하였다. 논문에서는 동일한 조건으로 CPU에서 ceres-solver로 계산 했을 때 1450ms 로 나왔다고 이야기하였다.

내가 코드를 돌렸을 때 ~350번 프레임 쯤에서 에러치가 1.5 pixel보다 낮아졌다. 저자이 따르면 0번째 프레임부터 여기까지의 시간을 재는 것이 맞다고 한다.

하지만 현재의 코드는 매 iteration마다 `engine.run` 함수가 들어가있고, 이는 반복 연산에 효율적인 방법은 아니다. 그렇기에 오버헤드가 많이 발생하여 훨씬 느리게 보이는 것이라고 저자는 이야기한다. 

저자는** `repeat()`함수를 사용해서 350 iteration을 하나의 `engine.run`으로 넣으면 문제가 해결될 것**이라고 했고, 추후에 코드를 수정해 공유해주겠다고 했지만... 아쉽게도 나는 IPU 테스트 기간이 끝나버렸다 ㅜㅜㅜ 저자를 믿는 수 밖에... 

저자의 말로는 **다음 달 쯤에 논문이 또 나온다고 한다**!!! 신나버려!

 