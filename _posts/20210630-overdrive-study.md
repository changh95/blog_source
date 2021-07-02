---
title: 기타 이펙터 - 오버드라이브 계열과 기초 이론 공부
date: 2021-06-30 22:34:42
mathjax: true
tags: 
- 일상
- Guitar
- 이펙터
categories: 
- [Life, Guitar]
excerpt: 소프트클리핑/하드클리핑, TS/BB/ 이해하기
---

## 드라이브가 걸리는 원리

원래 앰프의 본래 역할은 말 그대로 소리 신호를 증폭해주는 역할을 한다 (amplifier). 깔끔한 소리가 들어가면, 깔끔한 소리가 훨씬 크게 나야하는 것이 앰프의 역할이다.

하지만 어떠한 이유에서 소리 신호를 특정 임계치 이상 증폭시킬 경우 찢어지는 소리가 난다. 이러한 현상에는 여러가지 이유가 있는데 가장 많이 보이는 이유 2개를 설명한다.

- 앰프 내부의 회로가 더 큰 소리를 감당하지 못해서 클리핑이 나는 경우
- 스피커가 가용할 수 있는 소리보다 더 큰 소리를 낼 수 없어서 불안정하게 진동을 하기 때문에 소리가 찢어지는 경우

전자의 경우를 그림으로 설명하면 아래와 같다.

![](https://www.howtogeek.com/wp-content/uploads/2011/05/Clipping_waveform.jpg?trim=1,1&bg-color=000&pad=1,1)

신호이론 상 가용치보다 더 큰 신호를 받으면, 해당 앰프가 낼 수 있는 최대 임계치 밑으로 신호가 모이면서 saturation 현상이 발생한다. 

이 때 saturation이 되는 모양은 soft clipping / hard clipping의 두가지 다른 패턴으로 나눠지게 된다. Soft clipping은 파동의 형태가 원래의 모양과 비슷한 것을 볼 수 있고 (i.e. 파형 전체의 크기가 변하는 것), hard clipping은 peak/trough가 주로 깎이는 것을 볼 수 있다. 이렇게 급격하게 입력 소리 파형의 모양이 바뀌는 것일수록 스피커에서는 안정적으로 반응하기 힘들기 때문에 더 찢어지는 소리가 나기도 한다.

어찌되었건 이런 식으로 소리를 찌그러트리는 것을 '드라이브를 건다' 라는 표현을 쓴다. 

얼마나 드라이브가 걸리냐에 따라서 Overdrive / distortion 등으로 나뉘며, 드라이브 특유의 소리에 따라서 crunch, fuzz 등의 드라이브 타입이 나뉘기도 한다. 

각 타입에서도 제조사의 톤 특색에 맞춰서 이름이 있다.

보통 [블루스](https://youtu.be/T38v3-SSGcM) 등에서 들리는 정도의 드라이브 소리를 overdrive, [락/메탈](https://youtu.be/9d4ui9q7eDM)에서 들리는 정도의 드라이브 소리를 distortion이라고 한다.

기타 이펙터 중에는 overdrive / distortion 계열 이펙터들이 있는데, **실제로 앰프의 소리를 엄청나게 키우지 않고도 소리의 파형을 변형시켜 이와 같이 찢어지는 소리를 내게 해주는 장치**이다.

&nbsp;

## Bass / Mid / Treble의 영역

기타 이펙터나 앰프를 보면 Bass/Mid/Treble을 나눠서 소리를 조절할 수 있다. Bass는 낮은 소리 영역, Mid는 중간 소리 영역, Treble은 높은 소리 영역을 뜻하며, 각각 담당하는 스펙트럼 영역은 다음과 같다.

- Bass는 20~180 Hz
- Mid는 180~1200 Hz
- Treble은 1200~10,000 Hz

(다른 자료에서는 5000Hz부터 Treble이라는 말도 있다. 그냥 대충 이 언저리 쯤에서 mid~treble의 영역을 가르는 것 같다.)

![](https://i.pinimg.com/originals/a9/f5/75/a9f57539217d75396452b6e0f50c3c6e.png)

오버드라이브를 공부할 때 이러한 이유를 알아야하는 이유는 다음과 같다.

- 오버드라이브 이펙터마다 강조하는 음역대가 다르다
- 다양한 악기들과 합주를 할 때, 내 음역대의 상태에 따라 내 소리가 묻히거나 앞서게 된다.
  - 내가 리드기타라면 당연히 앞서야한다.
  - 내가 백킹기타라면 너무 앞서면 안된다.

내가 원하는 소리를 내기 위해서는 어떤 오버드라이브가 필요한지 알아야 할 필요가 있다. 기타샵에 가면 정말 많은 오버드라이브 이펙터들이 있는데, 어떤 이펙터가 어떤 종류의 소리를 내고 합주에는 어떤 영향을 끼치는지 알면 큰 도움이 된다.

&nbsp;

## TS 타입 (Tube Screamer)

<img src="https://sc1.musik-produktiv.com/pic-003470039xl/ibanez-ts9-tube-screamer.jpg" width="500" height="500" alt="[alternative_text]" title="[이예에~~]">

아마 가장 유명한 드라이브 계열은 TS 계열이지 않을까?

TS 계열 드라이브 이펙터들 중에서는 Ibanez에서 만든 TS9과 TS808 등이 가장 유명하고, 그 후의 제품들은 리이슈나 리메이크 또는 비슷한 소리를 가진 제품들이 많이 나왔다. TS계열 이펙터를 알아보는 하나의 방법은 '초록색 색깔' 이라는 점인데, 많은 TS계열 제품들이 TS808/TS-9이 가진 초록색 색깔을 그대로 가져가기 때문이다. [TS계열 이펙터들을 비교하는 영상](https://youtu.be/kshrctNxEQE?t=151)을 보면 초록색 이펙터가 많다 ㅋㅋ

오리지널 TS808 / TS-9의 드라이브 톤은 굉장히 빈티지한 드라이브 톤을 만든다. [Stevie Ray Vaughan](https://youtu.be/AbKe_nash-g)가 TS 계열의 드라이브를 적극 활용해서 블루스에 맞는 오버드라이브 톤을 만들어낸다. 하지만 이런 톤은 모던 음악과는 잘 어울리지 않기 때문에 일반적으로는 다른 계열의 드라이브를 사용한다.

그러면 TS계열은 왜 유명한것일까?

TS계열의 **진짜 면모는 '아주 약간의 드라이브'를 건 상태에서 부스팅을 할 때** 나타난다. [Eric Johson](https://youtu.be/hT9NhjXd9LM?t=98), [John Mayer](https://youtu.be/ncijv1WCnVk?t=59)과 같이 유명한 톤은 다 TS계열 부스팅의 덕을 보았다. Drive 노브는 12시를 넘기지 않고 Level 노브를 높혀서 드라이브가 살짝 묻은 상태로 볼륨을 높히는 것인데, TS계열 특성상 아래 소리 스펙트럼 그래프에서 보이는 것 처럼 **bass와 treble은 깎이면서 mid 볼륨이 엄청나게 커진다**. 이때 이 커진 mid를 **크런치나 퍼즈를 연결하면 후발 드라이브의 효과가 극대화**된다. 또, **mid가 커지면 합주를 할 때도 소리가 두각을 나타내기 때문에 리드기타 톤을 만들기 좋다**.

<img src="https://heartfx.net/ts9/frtonmax.png" width="500" height="500" alt="[alternative_text]" title="[이예에~~]">

물론 mid가 커진다고 모든 기타에서 좋은 소리가 나는 것은 아니다. 이미 mid가 빵빵한 기타 소리에 mid를 더 밀어주면 굉장히 듣기 거북한 소리가 나게 된다. 그렇기 때문에 보통 **스트랫이나 텔레처럼 mid가 많이 부족한 기타가 TS계열을 썼을 때 큰 효과를 받는 편**이다. 아주 niche한 예시로, [메사부기 앰프](https://youtu.be/1gmGBoZWxw4?t=239)를 사용하면서 메탈 음악을 할 때 TS계열 드라이브를 이용해서 mid를 펌핑하고 게인을 미는 경우가 있다.

TS계열 드라이브는 자글자글한 소리를 내는데, 이는 굵기가 확실하게 들리는 드라이브를 좋아하는 사람들에게는 좋지 않은데, **어떤 드라이브던지 자글자글한 드라이브 감을 추가**해버리기 때문이다.

&nbsp;

## KLON 계열 

<img src="https://online.berklee.edu/takenote/wp-content/uploads/2013/01/klon-centaur.jpg" width="500" height="500" alt="[alternative_text]" title="[이예에~~]">

위 사진의 이펙터는 KLON Centaur인데, 가격은 4천 달러를 넘어간다. 물론 저 가격이 말도 안된다고 생각하는 사람들이 많아서 소리를 카피한 훨씬 저렴한 이펙터들도 많다 (KLON의 클론... 후후...). 오죽하면 [블라인드 챌린지](https://youtu.be/8tjQXTMfxYY)도 있었다 ㅋㅋ

Hard clipping 드라이브라 컴프감이 들어가있는데, 이 **컴프감이 굉장히 자연스럽다**.

또, 드라이브에 적당히 자연스럽게 클린 시그널도 섞여들어가있는데, **강하게 치면 드라이브가 좀 더 섞이고 약하게 치면 또 부드러운 클린 섞인 소리가 나온다**.

TS계열이 미드부스팅 특화였다면, KLON은 **특색이 없는 게인 부스터**라고 볼 수 있다. Bass/Mid/Treble을 거의 인풋 그대로 가져가기 때문에 (i.e. 특색이 없기 때문에), **아무 앰프게인/ 오버드라이브 이펙터와 섞어도 자연스럽게 드라이브를 얹어줄 수 있다**. 이러한 점 때문에 인기가 많은 것 같다. [소리 샘플](https://youtu.be/Z0pSt5aGm_g)

&nbsp;

## BB 계열 (Blues Breaker)

<img src="https://images.reverb.com/image/upload/s--3f35PQ2W--/a_exif,c_limit,e_unsharp_mask:80,f_auto,fl_progressive,g_south,h_1600,q_80,w_1600/v1444947257/irgvvo2soqldx3spmhoa.jpg" width="500" height="500" alt="[alternative_text]" title="[이예에~~]">

KLON처럼 **특색이 없는 드라이브**라고 한다. 대신 다른 점이라면 **soft clipping 오버드라이브**라는 점이다.

KLON보다 좀 더 두드러지는 드라이브를 가지고 있는데, 오리지널 Marshall Blues breaker 이펙터는 게인량이 그렇게 크지 않다고 한다. 소프트한 블루스 이상의 드라이브를 얻기 어렵다고 하기 때문에, **최근 BB계열 이펙터들은 이러한 점을 개선해서 게인량을 늘렸다**고 한다. [소리 샘플](https://youtu.be/7ZS9SQBEotw?t=664)

하지만 소프트한 블루스에는 이것만큼 좋은게 없다는 이야기도 있는데, 존메이어의 백킹톤이 왠만하면 BB에서 나온다는 이야기가 있다 (리드톤은 아마 TS계열 부스트가 붙지 않을까 싶다). [소리 샘플](https://youtu.be/cYHXMazyhWI)