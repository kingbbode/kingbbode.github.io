---
layout: post
title: GDG - 측정하는 놈:그리는 놈:로딩하는 놈
categories: [seminar]
tags: [seminar]
fullview: false
comments: true
published: true
outlink: 0
---

세미나 정리 및 후기.

# **GDG WebTech Web**
## **측정하는 놈, 그리는 놈, 로딩하는 놈**
>2016년 8월 27일
---

## 랜더링 퍼포먼스

### GPU가 잘하는 것
 - 텍스텨 이미지 출력
 - 이미 가진 텍스쳐 재활용
 - 회전, 확대, 축소, 기울임, 반투명 처리
 - 위 기능 동시 처리

### GPU가 못하는 것
 - 비디오 메모리로의 데이터 전송 느림
 - GPU의 데이터는 CPU에서 생성 후 전송

**데이터 전송 최소화 필요**

### 이슈
 - Reflow & Repaint 발생 요인
    * DOM 노드의 동적인 crud
    * DOM 노드의 감춤/표시
    * DOM 노드의 이동, 애니메이션
    * 스타일시트의 추가 혹은 스타일 속성의 변경
    * 브라우저 사이즈 변경
    * 폰트 변경
    * 스크롤

---

## 로딩 퍼포먼스

#### JavaScript
 - defer : exec는 돔 그린 후에
 - async : load 후에 돔 정지하고 exec 후 돔 시작

#### Defer iFrames
 - data-src="" 활용

## 유용한 기능

#### Adding User-timing
window.performance.mark()
window.performnce.measure()

#### Chrome Dev Tool
- TimeLine, Profiles -> 자바스크립트API로 실행 가능.

#### performance Timing
- window.performance


---

### 후기
> 페이지 로딩속도를 개선하기 위한 로딩 퍼포먼스에 많은 기대를 하였다.
> 그러나 대부분의 내용이 렌더링 퍼포먼스의 내용이였고, 로딩 퍼포먼스 부분에서는
> async, defer에 대한 정말 간단한 내용이였기때문에 기대한 내용은 아니였다.  
> 시간이 부족하여 유용한 기능, 유용한 Tool 등의 소개는 대부분 생략되었다는 점이 정말 아쉽다. 
> 백엔드, 프론트를 개발하며 css는 다루지 않기때문에 로딩 퍼포먼스, 유용한 기능 & Tool 소개에서 
> 많은 기대를 하였었는데 이 부분에서 얻을 수 있는 점이 거의 없어서 아쉽다.


