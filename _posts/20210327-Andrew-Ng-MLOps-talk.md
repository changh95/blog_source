---
title: MLOps - From Model-centric to Data-centric AI (Andrew Ng 발표)
date: 2021-03-27 15:16:43
mathjax: true
tags: 
- MLOps
- Andrew Ng
- Deep Learning
categories: 
- [Deep Learning, 학회 발표 리뷰]
excerpt: Deep Learning AI - A Chat with Andrew on MLOps - From Model-centric to Data-centric AI 강의 정리
---

{% youtube 06-AZXmwHjo %}

&nbsp;

(Andrew Ng 교수님 최고... 상냥보스...)

## How to improve AI systems

- AI systems = Code (Model) + Data
  - Model도 중요하고 Data도 중요하다.
- 성능을 높이려면 두가지 방법이 있다.
  - 1. Model을 개선하기
    - SOTA 모델을 사용하기
    - Hyperparameter 튜닝
  - 2. Data를 개선하기
    - 데이터 품질 개선하기
  - Baseline 모델이 있을 때, 성능을 더 높여야한다면 어떤 방법을 먼저 사용하겠는가?
    - 교수님 팀의 실험으로는 **Data를 개선하는 것이 제일 효과적이였다고 한다**.

{% asset_img "code_vs_data.png" "Code vs data" %}

&nbsp;

---
## Data is food for AI

- 데이터 준비 과정 vs 데이터 사용 과정 (80:20)
  - 데이터는 준비하는 과정이 훨씬 더 오래 걸린다.
  - 모델이 데이터를 직접적으로 사용하는 부분은 전체 프로세스에서 굉장히 작은 부분이다.
    - 하지만 99%의 AI연구는 이 분야에 집중한다.

&nbsp;

---
## Lifecycle of an ML project

{% asset_img "code_vs_data.png" "Code vs data" %}

- Scope project
  - 문제를 구체화한다.
- Collect data
  - 데이터의 label이 제대로 되어있는가?
    - 데이터 label이 정확한지 어떻게 알 수 있는가? 어떻게 평가할 것인가? 
    - 어떤 특정 labelling 방식을 정하고, 모든 데이터셋에서 이 방식을 유지해야한다. 그렇지 않으면 데이터에 노이즈가 생긴다.
  - **데이터 품질을 일정하게 유지**해야한다.
    - 이것은 **systematic하게 만들자! == MLOps**
    - 데이터 labelling을 해주는 인력에게 샘플을 요구하자. Consistency를 확인해야한다.
- Train data
  - 트레이닝
  - Error analysis
  - 개선점 찾기
    - Model-centric? Data-centric?
- Deploy in production
  - Systematically check for concept drift / data drift
    - 고객이 원하는 수준은 항상 높아짐
  - Flow data back to retrain / update model regularly

{% asset_img "label.png" "Label" %}
{% asset_img "label2.png" "Label2" %}

&nbsp;

---
## Model-centric vs Data-centric

- **Model-centric view**
  - 데이터는 모을 수 있는 만큼 모은다.
  - 데이터 내부에 있는 노이즈에 잘 대응할 수 있는 모델을 만든다.
    - 데이터는 고정하고, 지속적으로 모델을 개선해나간다.
  - 모델 튜닝을 어떻게 해야 더 좋은 성능을 얻을 수 있을까?
- **Data-centric view**
  - 여러 툴을 이용해서 **데이터 품질을 높인다**.
  - 이 경우, **여러 모델을 동일한 데이터로 학습**할 수 있다.
    - 대신 모델을 고정하고, 데이터를 지속적으로 개선한다.
  - 데이터를 어떻게 개선해야 (e.g. 데이터 추가, augmentation, labelling) 더 좋은 성능을 얻을 수 있을까?
    - Augmentation, 데이터 추가 - Input x를 바꾸는 방법
    - Give more consistent definition for labels - Labels y를 바꾸는 방법  

&nbsp;

---
## Clean vs Noisy data

- Clean + Consistent한 데이터는 중요하다!

{% asset_img "clean.png" "Clean" %}

> 예시 1: 500개의 데이터가 있는데, 그 중 12%는 노이즈가 끼어있다. 이 때 뉴럴넷의 성능을 높이는 방법에는 두가지가 있다: 1. 노이즈를 제거하기, 2. 새로운 데이터 500개를 얻어낸다 (i.e. 트레이닝 데이터를 2배로 늘린다). 어떤 방식이 더 좋을까?

데이터셋에서 노이즈를 제거함으로써 얻는 이득을 트레이닝 데이터를 키우는 방식으로 얻어내려면, 거의 3배의 데이터를 모아야한다. 

이미 큰 데이터셋을 쓰고 있다면... 더 많은 데이터를 모으기는 쉽지 않다 :)

작은 데이터셋을 쓰고 있다면 데이터 품질과 툴은 더욱더 중요해진다.

{% asset_img "cleannoisy.png" "Clean vs noisy" %}

&nbsp;

---
## MLOps

{% asset_img "mlops.png" "mlops" %}

- AI software has a more iterative development process, compared to the traditional software development.

&nbsp;

---

> From 'big data' to 'good data'