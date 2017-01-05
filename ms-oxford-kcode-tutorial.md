---
layout: page
title: xwMOOC 딥러닝
subtitle: 정보과학교육연합회-R을 이용한 인공지능 튜토리얼
output:
  html_document: 
    keep_md: yes
  pdf_document:
    latex_engine: xelatex
mainfont: NanumGothic
---




> ## 학습 목표 {.objectives}
>
> * 본인 나이, 사진속 얼굴 나이, 기계가 판단하는 나이를 비교한다.
> * 인공지능 API 서비스의 다양한 면을 살펴본다.
> * 인공지능 API 코딩 실습을 통해 코딩교육의 방향을 (나름) 설정한다.


## 1. R을 이용한 인공지능 튜토리얼

- [소프트웨어교육 어디로 가야 하는가?, 등록사이트](http://163.239.199.187:8888/ws2016/)
- [한국정보과학교육연합회(KiSEF) 홈페이지](http://kcode.kr/)
- [학술대회, 4차 산업혁명 시대의 소프트웨어 교육 어디로 가야 하는가? 공지사항](http://kcode.kr/notice/761)

<img src="fig/kcode-tutorial.png" alt="R 인공지능 튜토리얼 일정" width="77%" />

## 2. 나이측정 인공지능 서비스 [^how-to-detect-face-algorithm]

[^how-to-detect-face-algorithm]: [How to Detect Faces in Image](https://www.microsoft.com/cognitive-services/en-us/face-api/documentation/face-api-how-to-topics/HowtoDetectFacesinImage)

<img src="fig/ai-product-service-raw-material.png" alt="인공지능 서비스" width="77%" />

### 2.1. 나이측정 앱서비스 

- [How-Old.net, How old do I look? #HowOldRobot](https://how-old.net/)

### 2.2. 나이측정 API 둘러보기

- [마이크로소프트 Cognitive Services - Face API](https://www.microsoft.com/cognitive-services/en-us/face-api)

### 2.3. 나이측정 API 개발 사용자 인터페이스

- **사전 준비물**
    - 구독 열쇠(subscription keys) : https://www.microsoft.com/cognitive-services/en-us/apis
    - 사진 url: [https://raw.githubusercontent.com/statkclee/2016-11-06-sogang/gh-pages/figure/angry-face.jpg](https://raw.githubusercontent.com/statkclee/2016-11-06-sogang/gh-pages/figure/angry-face.jpg)
    - 매개변수: **returnFaceAttributes** : Age,Gender,Smile,FacialHair,HeadPose,Glasses


- [마이크로소프트 Cognitive Services](https://www.microsoft.com/cognitive-services/en-us/documentation)
    - Face API &rarr; API reference &rarr; Open API Testing Console : [실습 웹콘솔](https://dev.projectoxford.ai/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236/console)

> ### 얼굴인식 중요정보(face landmark) {.callout}
> 
> <img src="fig/face-landmarks.jpg" alt="얼굴 중요식별자" width="57%" />


## 3. 코딩 


